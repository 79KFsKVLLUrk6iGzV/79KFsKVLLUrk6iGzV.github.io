# Constructors

Instead of using a function like init in the previous examples to handle initialization, we can define a special methods specifically for initialization.  These methods are known as *constructors*.

Constructors have several benefits:

- Since they explicitly define which methods are used for initialization users of the type don't need to figure out which method should be called to do the initialization.
- The constructor is automatically called once, so a programmer cannot forget to call the initializer.  Improperly initialized values can cause errors.
- Constructors cannot (normally) be called twice, which can sometimes cause serious problems such as memory leaks.

Let's look at an example constructor:

```c++
class Date {
public:
	Date(int month, int day, int year); // constructor
	// ...
};
```

Notice that a constructor is a special kind of method whose name matches the class name and for which there is no return type listed (not even void).

The constructor is called when creating an object.  Let's see some examples:

```c++
Date springBreakStart = Date(3, 9, 2015);
Date springBreakEnd(3, 13, 2015);
Date gradesDue = Date { 5, 13, 2015 }; // C++ 11
Date presidentsDayHoliday { 2, 16, 2015 }; // C++ 11
new Date(1, 1, 1970);
new Date { 1, 1, 1970 }; // C++ 11
```

Initialization that contains an = operator is known as *copy initialization*.  Initialization without an = operator is known as *direct initialization*.

When an object is created it must be created with the correct arguments to match one of its constructors (it could match a generated default constructor, we'll learn about default constructors soon).

Since we've defined a constructor that takes 3 integers for Date, none of the following will compile:

```c++
Date a;              // no matching (parameterless) constructor
Date b(1);           // no matching constructor
Date c(1, 1);        // no matching constructor
Date d("", "", "");  // no matching constructor
```

## Constructor Overloads

We can provide different constructor overloads following the same rules as function overloading.

```c++
class Date {
private:
	int day;
	int month;
	int year;
public:
	Date(int month, int day, int year);
	Date(char const * iso8601);
	// ...
};
```

Now Date has 2 constructors.  We can create a Date by passing in the month, day, and year as before, or we can create one passing a string formated using the [ISO 8601](http://en.wikipedia.org/wiki/ISO_8601) format.

For example:

```c++
Date f("2015-01-16");
```

As with functions constructors can have default arguments.

## Default Constructors

A constructor that has no parameters is called a *default* constructor.

Here's an example of a default constructor:

```c++
// Stack.hpp
#pragma once

class Stack {
public:
	Stack(); // default constructor (declaration)
};
```

```c++
#include "Stack.hpp"

Stack::Stack() { // default constructor (definition)
}
```

The default constructor is called if no arguments are specified:

```c++
Stack s1;
Stack s2 = Stack();
Stack s3 = {};       // C++ 11
Stack s4 = Stack{};  // C++ 11

Stack s5(); // oops, compiler thinks this is a function prototype!!
s5(); // so the compiler thinks we can call s5, would be linker error
Stack s6 = s5; // compiler error, can't conversion function pointer to a Stack!
```

### Default Constructor for Built-In Types

Built-in types are considered to have default constructors, but they are not invoked.

This behavior is mainly to improve performance in a few important cases.  For example:

```c++
char buffer[32 * 1024]; // make a 32 kb buffer, not initialized
```

By avoiding initialization of the buffer, performance can be slightly better.

If we want the memory to be initialized for us, we can use the default initializer using {}.  For example:

```c++
int a{};             // initializes to default (i.e. 0)
char buffer[32] {};  // initializes each element to default (i.e. 0)
```

### Generated Default Constructor

When there is no constructor written in a class the compiler will attempt to generate a default constructor that takes no arguments and simply calls the default constructor on each member variable.  If a constructor is specified, then the compiler will not generate a default constructor.

Consider the following:

```c++
// Pair.hpp
#pragma once

struct Pair {
	int first;
	int second;
};
```

```c++
// main.cpp
#include "pair.hpp"

int main() {
	Pair pair1;

	return 0;
}
```

Notice that we can still make an instance of Pair (pair1) even though we never defined a constructor for Pair.  The default constructor simply calls the default constructor for each member.  In our case the members are both of type int, and remember from the previous section that the default constructors for built-in types are not invoked.  This means the default constructor for Pair simply leaves first and second uninitialized.

Now let's change Pair to have a constructor that initializes first and second.

```c++
// Pair.hpp
#pragma once

struct Pair {
	Pair(int first, int second);
	int first;
	int second;
};
```

```c++
// Pair.cpp
#include "Pair.hpp"

Pair::Pair(int f, int s) {
	first = f;
	second = s;
}
```

```c++
// main.cpp
#include "pair.hpp"

int main() {
	Pair pair1;                // ERROR: no default constructor
	Pair pair2(42, 43);        // OK
	Pair pair3 = Pair(44, 45); // OK
	Pair pair4 = {};           // ERROR: doesn't match any constructor
	Pair pair5 = {1};          // ERROR: doesn't match any constructor
	Pair pair6 = { 1, 2 };     // OK, calls Pair::Pair

	return 0;
}
```

Notice that as soon as we define a constructor for our type the compiler will no longer generate a default constructor for us.  This makes sense.  In our example the programmer is saying that creating a pair requires passing in the 2 integers that compose the pair.  If the compiler still generated the default constructor then the programmer could not express the requirement that the pair needs to be created with its components.

If we did want both forms we can add a default ourselves:

```c++
// Pair.hpp
#pragma once

struct Pair {
	Pair();
	Pair(int first, int second);
	int first;
	int second;
};
```

```c++
// Pair.cpp
#include "Pair.hpp"

Pair::Pair() {
}

Pair::Pair(int f, int s) {
	first = f;
	second = s;
}
```

```c++
// main.cpp
#include "pair.hpp"

int main() {
	Pair pair1;                // OK
	Pair pair2(42, 43);        // OK
	Pair pair3 = Pair(44, 45); // OK
	Pair pair4 = {};           // OK
	Pair pair5 = {1};          // ERROR: doesn't match any constructor
	Pair pair6 = { 1, 2 };     // OK, calls Pair::Pair

	return 0;
}
```

#### Explicit Defaults

In C++11 we have another option.  We can simply tell the compiler that we still want the generated default constructor and have the compiler write it for us:

```c++
// Pair.hpp
#pragma once

struct Pair {
	Pair() = default;
	Pair(int first, int second);
	int first;
	int second;
};
```

```c++
// Pair.cpp
#include "Pair.hpp"

Pair::Pair(int f, int s) {
	first = f;
	second = s;
}
```

Our program will behave as in the previous example, but we let the compiler do some of the work for us.  In this example perhaps the savings is a bit trivial, but the ability to explicitly specify a default for a generated constructor can be very convenient in some more advanced scenarios we'll look at soon.

### When to Use Default Constructors

It only makes sense to have a default constructor if the type in question has a natural default value.  If we defined a String type, then a natural default could be an empty string.  For a stack, list, or queue the natural default is probably and empty version of the container.

Many types may not have any natural default.  For example, our Date class may not have a sensical default.

## Member Initialization

What if we want to initialize some member variables within a constructor?  In the previous example we took the parameters passed to the Pair constructor and used it to set the member variables first and second.  Technically this was not initialization, it was assignment.  Let's step back and understand some of the basics of initialization and assignment in C++.

### Initialization Basics

First let's think about const variables.  Since they are constant, they cannot change.  That means that while they can be initialized they cannot be assigned.  Consider the following:

```c++
int const a = 42;  // a is initialized to 42
a = 43;            // ERROR: cannot assign to a const
```

The = in the first line is allowed because it is considered to be initialization.  The = on the second line is not allowed because it is assignment.  Giving a variable its first value is initialization, while subsequently changing the value is considered assignment.  Although the operator is the same, the semantics are different.  That is why the first line is allowed while the second is not.

For const variables initialization isn't just allowed, it is mandatory.  We cannot create a const variable that is not initialized.

```c++
int const a; // ERROR: no initialization
```

Why this is disallowed should be pretty obvious.  Knowing that it is slightly faster to leave variables uninitialized, it isn't unreasonable that a language would have a means of creating uninitialized variables (even here most newer languages do not provide such an option).  This doesn't mean that we would ever want to *use* a variable that wasn't initialized.  It just means that we want to make an uninitialized variable, because we plan on assigning it somehow before it is used (e.g. reading a file).  If we were allowed to make uninitialized const variables, then we could make variables that would always be uninitialized and whose value would always be undefined.

References in C++ are quite similar.  References are basically pointers, but unlike pointers they cannot be null and are accessed via dot notation.

```c++
int const a = 42;  // a is initialized to 42
int const &b = a;  // b references a
```

### Constructors and Initialization

Coming back to class constructors, let's examine how this relates to object creation.  Consider the following:

```c++
class Person {
private:
	int age;
public:
	Person(int a) // initialization of members happens here
	{ // body of constructor, Person is already initialized at this point
		age = a;
	}
};

Person john(42); // create a john, initialized with Person::Person(42)
```

In the code above age is initialized before the body of the constructor runs.  The statement *age = a;* in the body of the constructor assigns age to a, it is not initialization.  Since we haven't done anything to specify how the initialization should happen C++ will simply do default initialization on each member, which for int does nothing.  At this point just take that as a description of how C++ works.  It will make more sense soon.

What if we decide that the next thing we need is a pair of People.  Let's make a new class that contains two People.

```c++
struct Pair {
	Person first;
	Person second;
	
	Pair(Person const &a, Person const &b) {
		first = a;
		second = b;
	}
};
```

Seems like this should be reasonable, however it will not compile.  The problem is that as stated earlier member variable initialization has already completed by the time the body of the constructor starts executing.  Since we haven't done anything to specify how initialization should happen C++ will simply do default initialization for each member.  The problem is that Person has no default constructor.

We'll have a similar problem if we attempt make one of our member variables const.  Remember from earlier that const variables can (and must) be initialized.

```c++
class Person {
private:
	Dna const dna;
public:
	Person() {
	}
};
```

Here we define a Person class that states that all instances of the Person class will have Dna.  The software assumes that a person's DNA cannot change after the person is created.  This is achieved by making the dna member const.  This will also cause an error.  Since dna is const it must be initialized.  Remember that initialization happens before the constructor body starts executing.

These examples all lead us to the point that we need to initialize member variables *before* the constructor body.  But how do we express this in C++?

### Member Initializer List

We can avoid all the previous problems as well as several others by properly initializing member variables by using *member initializer lists* instead of assignment.  Let's see how to initialize members properly in C++ classes:

```c++
class Person {
private:
	int age;
public:
	Person(int a)
		: age(a)  // member initializer list
	{
	}
};
```

We can initialize more than one variable by using comma separators:

```c++
class Person {
private:
	int age;
	int weight;
public:
	Person(int a, int w)
		: age(a), weight(w)  // member initializer list
	{
	}
};
```

Notice that the member variable comes first and then the value is specified inside parenthesis.  This means that we can even do something like this without ambiguity:

```c++
class Person {
private:
	int age;
	int weight;
public:
	Person(int age, int weight)
		: age(age), weight(weight)
	{
	}
};
```

#### Naming

Although that works, I typically use a prefix such as _ or m_ to indicate member variables.

How does this look when we separate our define the constructor outside the class definition:

```c++
// Person.hpp
#pragma once

class Person {
private:
	int _age;
	int _weight;
public:
	Person(int age, int weight);   // no member initializer list here
};
```

```c++
// Person.cpp

Person::Person(int age, int weight)
	: _age(age),            // member initializer
	  _weight(weight)       // list
{
}
```

#### const

Member initializer lists allow us to initialize constant member variables:

```c++
class Person {
private:
	Date const _dateOfBirth;
public:
	Person(Date const &dateOfBirth) : _dateOfBirth(dateOfBirth) {
	}
};
```

#### Initialization Order

One area for potential confusion is the order in which the members are initialized.  They are **not** initialized in the order they are listed in the member initializer list.  They are instead initialized in the order in which the members appear within the class defintion.  This typically isn't of much concern, but it can be if the members rely on each other for initialization:

```c++
class Foo {
public:
	int _a;
	int _b;
	Foo(int x) : _b(x), _a(_b + 1) { // BAD!!
	}
};

int main() {
	Foo f(42);
	cout << f._a << " " << f._b;
}

// output: <garbage> 42
```

Some compilers will give you a warning (many will not) if you member initializer list order differs from that of the member variables in the class definition.  Keeping the member variables and member initializer list in the same order is recommended to avoid confusion.

#### Members without Default Constructors

Member initializer lists also allow you to initialize member variables without default constructors.

```c++
struct Foo {
	int _a;
	Foo(int a) : _a(a) {
	}
};

struct Bar {
	Foo f;
	Bar() 
		: f(42) // calls Foo::Foo(int)
	{
	}
};
```

#### Performance

Member initializer lists can be faster.  This is because we avoid having both initialization and assignment.  Normally this isn't too big of a concern, but it can be.  As we'll see later, a types can have custom assignment operations that could be expensive.

#### Assignment vs Member Initializer Lists

Prefer member initializer lists vs assignment within the constructor body.  They are better in basically every way and there are no real downsides.  I might take off points if I see you using assignment instead of member initialization in your assignments.

#### Complex Initialization

Initialization logic can be as sophisticated as we want.  It is perfectly fine to call functions in member initializer lists.  We should avoid calling (non-static) methods, however because the object exists in a incompletely initialized state and this could cause bugs.  If you have complex initialization I would recommend placing it in a function within the cpp file (with internal linkage).

```c++
// Person.hpp
#pragma once

class Person {
private:
	int _age;
public:
	Person(int age);
};
```

```c++
// Person.cpp
#include "Person.hpp"

namespace {
	int thirtyIsTheNewTwenty(int const age) {
		if (age >= 30 && age < 40)
			return age - 10;
		return age;
	}
}

Person::Person(int age) : _age(thirtyIsTheNewTwenty(age)) {
}
```

## explicit Constructors

Constructors invoked with a single argument have a powerful and potentially confusing ability to perform an implicit conversion.  As an example consider the following:

```c++
Date d = "2015-01-18"; // calls Date::Date(char const *)
```

Or perhaps even more surprisingly:

```c++
void foo(Date const d) {
}

int main() {
	foo("2015-01-18");
}
```

When foo is called, it causes an implicit call to Date::Date(char const \*), which creates an instance of Date.  That Date is then passed to foo.  This is called a *converting constructor*.

At times this can be extremely useful.  Consider the following:

```c++
#include <string>
using namespace std;

void foo(string const &name) {
}

int main() {
	foo("Johnny Appleseed");
	return 0;
}
```

We can call foo passing a string literal (char const *const) to a function taking the string class (std::string), because std::string has a constructor taking char const *const.

Unfortunately this behavior can be confusing as well.

Consider the following:

```c++
class Date {
	// ...
public:
	// ...
	Date(int year);
	// ...
};

void foo(Date const &date) {
}

int main() {
	foo(42); // works, but probably really confusing
	return 0;
}
```

Most likely it is just wrong and causing a bug.  If it was intended it is too obscure.

There is a way to fix this.  We can use the keyword explicit to prevent an implicit conversion.

```c++
class Date {
private:
	// ...
	int year;
public:
	// ...
	explicit Date(int year);
	// ...
};


Date::Date(int y) {
	year = y;
}

void foo(Date const &date) {
}

int main() {
	foo(42); // ERROR: cannot convert argument 1 from 'int' to 'const Date'
	return 0;
}
```

Note that explicit needs to be on the declaration, but is not allowed on the definition.

Unfortunately this is a case where the C++ defaults are wrong.  Constructors should be explicit by default unless the implicit behavior is desired in which case they should be marked implicit.  Unfortunately this is not how the language is defined and making a change like that would break way too much existing code to be practical.

This leaves us with the unfortunate rule that we should mark all constructors explicit unless there is a good reason for the implicit conversion to be allowed.  If the implicit conversion is desired a comment should be left to indicate as such.  Realistically this rule is frequently not followed.

<!--
**TODO:** multi-param explicit ctors
-->

<!--
**TODO:** inline definitions
-->