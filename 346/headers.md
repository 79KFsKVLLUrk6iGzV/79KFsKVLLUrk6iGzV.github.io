---
title: Headers
---
# Headers

## Motivation

When going through the [C/C++ compilation model](compile-model.html), we looked at how the preprocessor worked and some errors that can occur during the preprocessor stage of the build process.  Now we'll examine headers in a little more detail.  Hopefully this is all review, but if not don't worry it will sink in eventually.

Let's say that we've defined some really useful function called inc.  In fact it is so useful that we're using it in several places in our program.

```c++
// inc.cpp
int inc(int const a) {
	return a + 1;
}
```

```c++
// add.cpp
int inc(int const a);

int add(int const a, int const b) {
	int sum = a;
	for (int i = 0; i < b; ++i)
		sum = inc(sum);
	return sum;
}
```

```c++
// main.cpp
int inc(int const a);
int add(int const a, int const b);

int main() {
	return add(inc(1), inc(1));
}
```

```
[]$ g++ inc.cpp add.cpp main.cpp
[]$ ./a.out
[]$ echo $?
4
```

In this simple program we don't use any headers at all and everything works just fine, but there is a problem.  The declaration of the inc function is copied in both add.cpp and main.cpp.

This kind of copying is dictated by the C/C++ compilation model.  Remember every translation unit is compiled independently and all symbols be declared (or defined) before they are used.  This means there is going to be a lot of copying going on for large programs.

Let's say that we want to change the inc function from our example.  Now we want the inc function to work as follows:

```c++
// inc.cpp (altered)
int inc(int const a, int const increment = 1) {
	return a + increment;
}
```

Making this change will require editing every file in our program.  In the real world these problems are going to be much more difficult.  Writing real C, and even more so C++ programs, without using the preprocessor would be practically speaking impossible.

Let's update our original program to make use of headers.  We'll separate the declarations into separate header files.

```c++
// inc.hpp
int inc(int const a);
```

```c++
// inc.cpp
#include "inc.hpp"

int inc(int const a) {
	return a + 1;
}
```

```c++
// add.hpp
int add(int const a, int const b);
```

```c++
// add.cpp
#include "inc.hpp"

int add(int const a, int const b) {
	int sum = a;
	for (int i = 0; i < b; ++i)
		sum = inc(sum);
	return sum;
}
```

```c++
// main.cpp
#include "inc.hpp"
#include "add.hpp"

int main() {
	return add(inc(1), inc(1));
}
```

```
#output
[]$ g++ inc.cpp add.cpp main.cpp
[]$ ./a.out
[]$ echo $?
4
```

When we compile the program we only need to pass in the .cpp files.  Remember header files aren't compiled directly.  Header files get included into translation units that are compiled independently.  Notice that we only passed the .cpp files to g++.  If we did pass the header files to g++ they would simply be ignored.

Also note the file extension.  An extension of .h is common for C programs.  C++ commonly uses the .hpp, .h, or .hxx extensions.  C source files should have the .c extension.  C++ source files should use .cpp, .cxx, or .cc.  Which extension you use doesn't matter, but they should be kept consistent within a single program.

Let's ask gcc to give us the preprocessor output from main.cpp:

```
[]$ g++ -E main.cpp
```

```c++
# 1 "main.cpp"
# 1 "<built-in>"
# 1 "<command-line>"
# 1 "main.cpp"


# 1 "inc.hpp" 1


int inc(int const a);
# 4 "main.cpp" 2
# 1 "add.hpp" 1


int add(int const a, int const b);
# 5 "main.cpp" 2

int main() {
 return add(inc(1), inc(1));
}
```

Side note: What are those weird # num things in the preprocessor output?  That's something called [linemarkers](https://gcc.gnu.org/onlinedocs/cpp/Preprocessor-Output.html).  They are used to tell gcc where the code actually came from (for error reporting and debugging).  They can be removed by using the -P g++ flag.

Aside from the linemarkers, the preprocessor output looks identical to our original program.  The key difference is that if we need to change the declaration of inc, our change is automatically updated throughout the program.

The preprocessor is basically creating the original verison of our program for us automatically.  This brings us to the key of understanding the preprocessor.  The preprocessor can be thought of as a code generator that can be used to automate the process of copying declarations throughout a program.

### Why Multiple Source Files Anyway?

Why do we need multiple source (.c/.cpp) files anyway?  If we simply have a single implementation file for our entire program can't we just ignore this whole mess with includes?  It is possible, but undesirable for several reasons.

- Organization: Breaking up a program into different source files containing related functionality typically makes the code easier to work with.
- Build Speed: If the entire program is a single source file then incremental builds are impossible.  Any change, no matter how trivial, will cause the entire program to be recompiled.  For large programs the wait time will be intolerable.
- Teams: When working projects with other programmers it is *much* easier if the program is divided into separate files.  Whenever two programmers make changes to the same file those changes need to be merged.  By breaking functionality into different source files the need for merging changes is greatly reduced.

**Don't put everything into a single source file**

## Guards

Include guards are a mechanism to prevent a header file from being included more than once.

### Why are guards needed?

As long as you only had declarations in header files, you technically wouldn't need include guards.  Remember it is perfectly fine to declare something more than once.  That being said there are many reasons why you might need to put a definition within a header file.  A common example is a class definition.

```c++
// Engine.hpp
class Engine { // definition of Engine
};
```

```c++
// Car.hpp
#include "Engine.hpp"

class Car { // definition of Car
	Engine _engine; // defines non-static data member; a Car has an Engine
};
```

```c++
// Truck.hpp
#include "Engine.hpp"

class Truck { // definition of Truck
	Engine _engine; // defines non-static data member; a Truck has an Engine
};
```

```c++
// driver.cpp
#include "Car.hpp"
#include "Truck.hpp"

int main() {
	Car const c();
	Truck const t();

	return 0;
}
```

```
# output
[]$ g++ driver.cpp
In file included from Truck.hpp:2,
                 from driver.cpp:4:
Engine.hpp:3: error: redefinition of ‘class Engine’
Engine.hpp:3: error: previous definition of ‘class Engine’
```

Examining the preprocessor output:

```
[]$ g++ -E -P driver.cpp
```

```c++
class Engine {
};
class Car {
 Engine _engine;
};
class Engine {
};
class Truck {
 Engine _engine;
};
int main() {
 Car const c();
 Truck const t();
 return 0;
}
```

So how do we fix this problem?  First let's look at the wrong way to fix this problem.

Since the problem is caused by something getting included twice through an header including another header, we might conclude that we shouldn't include anything from a header file.  Let's map out how this would look.

```c++
// Engine.hpp
class Engine { // definition of Engine
};
```

```c++
// Car.hpp
class Car { // definition of Car
	Engine _engine; // defines non-static data member; a Car has an Engine
};
```

```c++
// Truck.hpp
class Truck { // definition of Truck
	Engine _engine; // defines non-static data member; a Truck has an Engine
};
```

```c++
// driver.cpp
#include "Engine.hpp"
#include "Car.hpp"
#include "Truck.hpp"

int main() {
	Car const c();
	Truck const t();

	return 0;
}
```

In fact this does work fine, but it leads to some serious maintenance problems.  Let's assume that the Engine, Car, and Truck classes were written by Bob.  Let's also assume that they are real world classes, so they are quite a bit more complex than this simple example.  Now the classes are being used by Fred.  Now Fred knows what a Car is, but he has no idea about engines.  So imaging Fred tries to use the Car class:

```c++
// fred.cpp
#include "Car.hpp"

int main() {
	Car const c();

	return 0;
}
```

He tries to compile:

```
[]$ g++ fred.cpp
In file included from driver.cpp:4:
Car.hpp: error: ‘Engine’ does not name a type
```

Oops...now he's not sure what to do.  He does some looking around through the source or bugs Bob and finally he figures out the problem, he needs to include Engine.hpp before using Car, so he updates the program:

```c++
// fred.cpp

#include "Engine.hpp"
#include "Car.hpp"

int main() {
	Car const c();

	return 0;
}
```

and tries to compile again...

```
[]$ g++ fred.cpp
In file included from driver.cpp:4:
Engine.hpp: error: ‘SparkPlug’ does not name a type
```

Oops...I guess the Engine needs a SparkPlug (remember this is a more complex real world set of classes).  Well this might continue on until 20 more things end up getting included in.  At this point Fred is pretty mad at Bob for sending him on a wild header chase all afternoon.

Now imagine that Car is used in 100 different places in the program, but at this point we realize that the car also needs a new SelfDriver member (to implement the new self-driving feature).  Now we need to go back an update all 100 places car is used to have it include "SelfDriver.hpp".

This might seem like a made up example, but I promise that this has happened to me way more often than it should.

So we do want to include headers from within a header.  This serves as a way to document the dependencies of a header.  In our case Car was dependent on Engine, so Car should include Engine.  Another way of thinking about this is that each header file should compile correctly if included in a source file that *only* includes that header.

### How Do Guards Work?

There are two basic mechanisms to implement include guards.

#### Include Gaurds

```c++
// Engine.hpp
#ifndef ENGINE_HPP_INCLUDED
#define ENGINE_HPP_INCLUDED

class Engine { // definition of Engine
};

#endif
```

The first time Engine.hpp is included within a translation unit ENGINE_HPP_INCLUDED will not be defined, so the main body of the header will be processed the first time.  The next time Engine.hpp is included within that translation unit ENGINE_HPP_INCLUDED will already be defined (because of the #define ENGINE_HPP_INCLUDED line) so the main body of the header will be skipped.

#### Pragma Once


```c++
// Engine.hpp
#pragma once

class Engine { // definition of Engine
};
```

Pragmas are special instructions to the preprocessor or compiler.  In this case the once pragma tells the preprocessor to only allow this file to be included once per translation unit.  Generally pragmas are specific, so they may not work on every compiler.  That said, pragma once is a widely supported directive that works on all major compilers.

Personally I strongly prefer the pragma once approach.  The include guard has many disadvantages:

- Requires a unique name.
   - Generally this is a filename, but filenames within a project may not be unique within a large project.  
   - Files are frequently copied and the guard names might not be updated.
   - Updating the filename correctly requires updating the filename, and two places in the file.  The more places that need to be updated the less likely it will be done correctly.
- Requires more lines of code (3 lines vs 1)
- Greater cognitive load (preprocessor state)
- Imperitive vs Declaritive (include guards are code that specify what to do, pragma once is an instruction stating what you want done)
- Sometimes can be slower than pragma once (the compiler can sometimes do better optimizations with pragma once)

I would recommend using pragma once unless it is likely you need to target a compiler that doesn't support it.

### When to Use Guards

Always use include guards in every header.  Even if you know you don't need guards right now, just add them anyway.  They won't hurt anything if they're there and not strictly needed.

## What to Include

Each header should include everything necessary for that header to be compiled and nothing more.

For now we'll start with the following rules:
1. Every header should be able to be included from an otherwise empty source file and compile.
2. Every include from a header should be required, i.e. if that include were removed rule #1 would be violated.

For now we can start with these basic rules.  Eventually we'll learn additional techniques that will allow us to further reduce what needs to be in our headers to meet the first two rules.

**Why do we need the second rule?  Everything seems to work fine even if I include too much.**

The reason we want to minimize what gets included in our headers is mainly due to compilation speed.  As programs grow larger compilation times can quickly get out of control.  Since a header file may be included in many source files, and change to a header will require all source files that include that header (directly or indirectly) to be recompiled.

The worst case would be a program in which every source file included the same include file and that include file included all other include files.  At this point whenever any include file changes, everything needs to be recompiled.

**TODO:** Additional rules for covering specifics of classes, derived classes, fwd references to classes, interfaces, circular dependencies, templates, inlining

**Example**

```c++
// Car.hpp
#pragma once

#include "Engine.hpp"
#include "Truck.hpp" // not needed, violates rule #2

class Car {
	Engine _engine; // uses Engine, needs to include it
};
```

If the following compiles we're passing rule #1.  If we can't comment out any more #include lines from the header and still compile then we're passing rule #2.

```c++
// test.cpp
#include "Car.hpp"
```

