---
title: Linkage
---
# Linkage

## Declaring vs Defining

C/C++ require that all symbols be declared (or defined) before they are used.  The *declaration* gives the compiler the information it needs to reference that name.  Let's look at an example declaration:

```c++
double pi(); 
```

The previous declares a function named pi.  This tells the compiler that there is a function named pi which takes no arguments and returns a float.  Once we've declared pi the compiler will let us call it.

A *definition* provides and implementation of the name.  This definition is needed by the linker.  Let's look at the defintion of pi.

```c++
double pi() {
	return 3.14159265359;
}
```

C/C++ allow program entities to be declared more than once, but it can only be defined once.  If something is defined 0 times then it is an undefined reference.  If something gets defined more than once then it is a multiply defined symbol.

## Linkage

A name that refers to some program entity (function, variable, type, etc.) may have *linkage*.  Linkage describes how entities from one translation unit can be linked to another translation unit by the linker.

First let us consider two types of program entities:

- local: defined within a function or block
- non-local: defined at file scope

If a program entity is local then it always has no linkage--it is not accessible from another translation unit.  Non-local program entities can be accessed more broadly.  But how broadly can they be accessed?  This is defined by linkage.

Consider the following types of linkage:

- *no linkage*: The name can be referred to only from the scope it is in.
- *internal linkage*: The name only refers to an entity within the same translation unit.
- *external linkage*: The name may refers to an entity shared among translation units.
   - *language linkage*: The name be shared with translation units written in another programming language.

<table>
  <tr>
    <td rowspan="2">If the same name exists in multiple translation units with</td>
  
    <td>internal</td>
    <td rowspan="2">linkage then each name refers to</td>
    <td>a different</td>
    <td rowspan="2">program entity.</td>
  </tr>
  <tr>
    <td>external</td>
    <td>the same</td>
  </tr>
</table>

In C/C++ linkage is controlled via 3 mechanisms:

- **static** keyword: specifies internal linkage
- **extern** keyword: specifies external linkage
- anonymous namespace (C++ only): specifies internal linkage (this is the recommended approach for C++ programs)

If you don't explicitly specify linkage then the default linkage will be used for that program entity.  What that default is depends on the type of program entity.  If you don't use any of the previous mechanisms to explicitly control the linkage than the default mechanism will be used.

Default Linkage of various program entities:
- no linkage: local variable, function parameter, local types
- internal linkage: non-local constant variables
- external linkage: functions, non-local mutable variables, types (and their members)

Hopefully some examples will make things more clear:

### Function with external linkage

Let's start with a simple example. Remember functions have external linkage by default.  Let's define a function in one .cpp file and call it in another:

foo.cpp

```c++
int foo() { return 42; } // define foo, has external linkage
```

main.cpp

```c++
// declare foo, we need to declare before we can call it
int foo();

// define main, has external linkage
int main() { return foo(); } 
```

output:

```
[]$ g++ foo.cpp main.cpp
[]$ ./a.out
[]$ echo $?
42
```

### Mutable Non-Local Variable

For our next example let's use a non-local variable.  Remember that mutable non-local variables have external linkage by default.

foo.cpp

```c++
int callCount; // define callCount, has external linkage
void foo() { ++callCount; }
```

main.cpp

```c++
// declare callCount, we need to declare it before we can use it
extern int callCount; // use 'extern' so we declare the variable

void foo();

int main() {
	callCount = 0;
	foo();
	foo();
	return callCount;
} 
```

output

```
[]$ g++ foo.cpp main.cpp
[]$ ./a.out
[]$ echo $?
2
```

### Named Non-Local Constant

Next let's examine non-local named constant.  Remember they have internal linkage by default.

inc.cpp

```c++
// define increment, has internal linkage
int const increment = 1;

int inc(int const input) {
	return input + increment;
}
```

main.cpp

```c++
int inc(int const);

int main() {
	return inc(1);
}
```

output:

```
[]$ g++ inc.cpp main.cpp
[]$ ./a.out
[]$ echo $?
2
```

What if we change main.cpp to try to use increment?

main.cpp (altered)

```c++
#include <iostream>

int inc(int const);

// declare increment
extern int const increment;

int main() {
	std::cout << "increment: " << increment << std::endl;
	return inc(1);
}
```

output:

```
[]$ g++ inc.cpp main.cpp
/tmp/ccxxRVrI.o: In function `main':
main.cpp:(.text+0xb): undefined reference to `increment'
collect2: ld returned 1 exit status
```

This produces a linker error.  Since increment is defined in inc.cpp with internal linkage it cannot be referenced in another translation unit such as main.cpp.  We can fix this problem by changing increment to use external linkage.

inc.cpp (altered)

```c++
// define increment, has external linkage
extern int const increment = 1;

int inc(int const input) {
	return input + increment;
}
```

output:

```
[]$ g++ inc.cpp main.cpp
[]$ ./a.out
increment: 1
[]$ echo $?
2
```

### Function with Internal Linkage

For our next example, let's make a function that uses internal linkage.  Remember the default linkage for functions is external, so we have to use the keyword static or an anonymous namespace.

add.cpp

```c++
// define inc with internal linkage
// this inc is only accessible for linking in this translation unit
static int inc(int const value) {
	return value + 1;
}

int add(int const a, int const b) {
	int sum = a;
	for (int i = 0; i < b; ++i)
		sum = inc(sum);
	return sum;
}
```

main.cpp

```c++
// declare add (from add.cpp)
int add(int const a, int const b);

// using an anonymous namespace is the preferred approach for C++
namespace // anonymous namespace
{
	// define inc with internal linkage (inside anonymous namespace)
	// this is a different inc than in add.cpp
	int inc(int const value) {
		return value + 10;
	}
}

int main() {
	return add(inc(2), inc(3));
}
```

output:

```
[]$ g++ add.cpp main.cpp
[]$ ./a.out
[]$ echo $?
25
```

### Internal Linkage for Mutable Non-Local Variable

For our final example, let's use internal linkage for a mutable non-local variable.  Remember that the default linkage for mutable non-local variables is external so we'll have to use either the static keyword or an anonymous namespace.

inc.cpp

```c++
// define callCount, has internal linkage
static int callCount;

int inc(int const a) {
	++callCount;
	return a + 1;
}

void initCallCount() {
	callCount = 0;
}

int getCallCount() {
	return callCount;
}
```

main.cpp

```c++
#include <iostream>
using namespace std;

// declare some stuff from inc.cpp
int inc(int const a);
void initCallCount();
int getCallCount();

// define callCount, has internal linkage, this is defines a new variable
static int callCount;

int main() {
	callCount = 42;

	initCallCount();
	inc(1);
	inc(2);

	cout << "callCount (in main.cpp): " << callCount << endl;
	cout << "callCount (in inc.cpp): " << getCallCount() << endl;

	return 0;
}
```

output:

```
[]$ g++ inc.cpp main.cpp
[]$ ./a.out
callCount (in main.cpp): 42
callCount (in inc.cpp): 2
```

### Summary

- Names must be defined or declared before they are used
- Everything should be defined exactly one
- static, extern, and anonymous namespaces specify linkage

```c++
void a() {}               // defines a, has external linkage
void a();                 // declares a

extern void b() {}        // defines b, has external linkage
void b();                 // declares b

static void c() {}        // defines c, has internal linkage
void c();                 // declares c

namespace { void d() {} } // defines d, has internal linkage
namespace { void d(); }   // declares d

int e;                    // defines e, has external linkage
extern int e;             // declares e

static int f;             // defines f, has internal linkage

namespace { int g; }      // defines g, has internal linkage

int const h = 1;          // defines h, has internal linkage

extern int const i = 1;   // defines i, has external linkage
extern int const i;       // declares i
```
