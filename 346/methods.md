---
title: Methods
---
# Methods

Methods define operations that an instance of a class can perform.

## Separate Operations/Data

Suppose we have a struct as follows:

```c++
struct Date {
	int day;
	int month;
	int year;
};

// initialize a Date
void init(Date &date, int month, int day, int year);

// adds yearCount years to a date
void addYear(Date &date, int yearCount);
// ...
```

init and addYear operate upon instances of the Date struct, but they have no real connection to Date.

## Combined Operations/Data

The previous style is common in C, but in C++ we can move the functions within the definition of Date to make the connection between the functions and Date explicit.

> Some languages like Java or C# require that *all* operations be defined as part of a class.  C++ places no awkwardness upon us if we *don't* want to explicitly connect data and operations together into a type.  IMO this is a good thing, not all problems call for the same solution.

Let's look at how C++ allows us to combine data and operations together in the same type:

```c++
// Date.hpp

struct Date {
	int day;
	int month;
	int year;

	void init(int month, int day, int year);
	void addYear(int yearCount);

	// ...
};
```

## Member Access

Now init and addYear are *methods* (or member functions) of Date.  To invoke these methods we use the same member access syntax as when accessing structs (dot, or arrow):

```c++
Date linusTorvaldsBirthday;
linusTorvaldsBirthday.init(12, 28, 1969);

Date sethMeyersBirthday = linusTorvaldsBirthday;
sethMeyersBirthday.addYear(4); //1973

Date *stanLeeBirthday = new Date;
*stanLeeBirthday = sethMeyersBirthday;
stanLeeBirthday->addYear(-51); // 1922
delete stanLeeBirthday;
```

## Method Definition

Each class/struct acts as a unique namespace for its members.  This means that different types can have methods with the same name and signature.  This means that we must specify the type name when defining members.

```c++
// Date.cpp

void Date::init(int m, int d, int y) {
	month = m;  // which month? Date::month
	day = d;    // which day?   Date::day
	year = y;   // which year?  Date::year
}

void Date::addYear(int yearCount) {
	year += yearCount;
}
```

Ignoring Date::, init would look just like the definition of the C init function above.  Date:: simply sepecifies that we're talking about the init function that is part of Date.  Reading backward :: basically reads 'belonging to', i.e. init is a function belonging to Date.

## Member Variables

Members of a type (e.g. member variables, and methods) can be referenced from within that class's methods without explicitly referencing an object.  For example, inside the init method, month, day, and year are all referred to without an explicit object reference.

Each instance has its own copy of member variables (except for static member variables which we will get to later).  This works exactly the same as with structs in C.  By default when a method references a member variable, it references that instance's member.


```c++
// Person.hpp
#pragma once

struct Person {
	int age;

	void setAge(int a);
};
```

```c++
// Person.cpp
#include "Person.hpp"

void Person::setAge(int a) {
	age = a;
}
```

```c++
// main.cpp
#include <iostream>
#include "Person.hpp"

using namespace std;

int main() {
	Person john;
	john.setAge(1);     // sets john.age = 1

	Person maryAnn;
	maryAnn.setAge(2);  // sets maryAnn.age = 2

	cout << john.age << " " << maryAnn.age << endl;

	return 0;
}

// outputs: 1 2
```

In main we made 2 instances of the Person struct, so that means there are 2 instances of Person::age.  Changing john's age does not change maryAnn's age.

We can think of member functions knowing which instance they were called from.  This is possible because each member function has an implicit argument that is the object from which the method was invoked.  We will cover this more later, but for now we can think of this very much like the examples from the beginning of this section, but instead of passing a reference explicitly the reference is passed automatically for us.  This is something C++ does for us automatically that we would have to do manually in C.

## Scope

The scope that member variables exist within is known as *class scope*.

### Name Hiding (Shadowing)

C++ names have scopes that define what parts of program a name may be accessed.  Scopes can be nested causing a name defined in a parent scope to be hidden (or shadowed).

Consider the following:

```c++
int const a = 42;
cout << a << " ";

{
	int const a = 43;
	cout << a << " ";
}

cout << a << endl;

// outputs: 42 43 42
```

### 

A class defines its own scope.  Member variables exist within this scope.  Class methods define their own function scope, which is a child scope of the class scope.  For example:

```c++
// Foo.hpp

#pragma once

struct Foo {
	int a;
	void method1();
	void method2();
	void method3(int a);
};
```

```c++
// Foo.cpp

#include "Foo.hpp"

void Foo::method1() {
	cout << a << " ";
}

void Foo::method2() {
	int const a = 2; // hides Foo::a
	cout << a << " ";
	{
		int const a = 3; // hides first local a
		cout << a << " ";
	}
}

void Foo::method3(int a) { // hides Foo::a
	cout << a << " ";
}
```

```c++
// main.cpp

#include "Foo.hpp"

int main() {
	Foo f;
	f.a = 1;
	
	f.method1();
	f.method2();
	f.method3(2);

	return 0;
}

// output: 1 2 3 1
```

