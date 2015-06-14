# Class Basics

Let's start to look at how to use classes and objects.  At first we will consider concrete types.  These types are designed to model values.  This is called class-based or value-oriented programming.

## Introduction

- Classes are a way of defining user-defined types that can be used in the same way as built-in types.
- Types define a specific representation of a concept.  The built-in type int, for example, is an approximation of the mathematical integers.
- We could define our own types to represent concepts not found in a built-in type (e.g. Uri).

Why define types?  Generally, well defined types allow for programs that are:

- More understandable--reads/writes at a higher level of abstraction
- Easier to modify--one place defines an entire concept
- More reliable--the compiler can help detect more problems, the type should define proper usage

## Quick Summary of Classes

- A class is a user-defined type.
- A class has members.  Member variables (data members) and methods (member functions) are the most common members.
- Classes define the meaning of creation (constructor), copy, move (C++11), and cleanup (destructor).
- Members are accessed with . (objects) and -> (pointers)
- Classes can define the meaning of operators (e.g. +, !, [])
- A class defines a namespace for its members.
- Access specifiers (e.g. public, private) allow for a separation of interface (how the type may be used) and implementation (how it works).
- A struct is a class whose members are public by default (class members are private by default).

Example:

```c++
class Person {
private:		// implementation details are private
	int age;	// defines member variable of type int named age
public:
	Person(int a) : age(a) { // constructor, initializes age to a
	}

	void setAge(int a) {	// method (member function)
		age = a;			// sets the member variable
	}
};

void foo(Person person, Person *personPtr) {
	person.setAge(40);		// call method with . (dot)
	personPtr->setAge(99);	// call method with -> (arrow)

	person.age = 19;		// illegal: cannot access private member
}

int main() {
	// create a variable of type Person named person1, intialized to 20
	Person person1(20);

	// create a variable of type Person named person1, intialized to 21
	Person person2(21);

	foo(person1, &person2);

	return 0;
}
```

Now that we've seen a basic example of classes, let's pull back a bit and build up an understanding of classes from the ground up.

## Methods

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

This style is fairly common in C, but in C++ we can move the functions within the definition of Date to make the connection between the functions and Date explicit.

```c++
struct Date {
	int day;
	int month;
	int year;

	void init(int month, int day, int year);
	void addYear(int yearCount);

	// ...
};
```

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

Each class/struct acts as a unique namespace for its members.  This means that different types can have methods with the same name and signature.  This means that we must specify the type name when defining members.

```c++
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

Members of a type can be referenced from within methods without explicitly referencing an object.  For example, inside the init method, month, day, and year are all referred to without an explicit object reference.

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

## Default Copying

By default objects support copying.  The default way of copying an object is to simply copy each of its members.  We can disallow copying or customize what happens during a copy, but we'll get to that later.

We already saw an example of copying:

```c++
Date sethMeyersBirthday = linusTorvaldsBirthday;
```

C++ supports several syntaxes for copying:

```c++
Date sethMeyersBirthday(linusTorvaldsBirthday);
```

and in C++ 11 we can do this (part of uniform initialization):


```c++
Date sethMeyersBirthday { linusTorvaldsBirthday };
```

or:

```c++
Date sethMeyersBirthday = { linusTorvaldsBirthday };
```

Note that this is the same syntax we use for primitive types:

```c++
int a = 42;
int b(42);
int d{42};    // C++ 11
int e = {42}; // C++ 11
```

We can also do assignment.  By default assignment does memberwise assignment, but this too can be disabled or overridden.

```c++
Date snlParty;
snlParty = sethMeyersBirthday;
```

## Access Control

With our Date struct there are no restrictions on how it can be used externally.  We might want to specify an interface that defines how our type may be used.  This interface can hide the specifics of how our type is implemented.

With structs everything is *public* (externally accessible) by default.  Take the following example:

```c++
Date linusTorvaldsBirthday;
linusTorvaldsBirthday.init(12, 28, 1969);

linusTorvaldsBirthday.day = 4000;
```

Clearly 4000 isn't a valid day.  Since day is public, code using Date is free to modify any of its members at will.

There is another access modifier called *private* that makes class members hidden from outside the class.  Private members are still accessible from inside the type, but cannot be accessed externally.

Let's change Date to restrict access to its implementation details:

```c++
struct Date {
private:
	int day;
	int month;
	int year;
public:
	void init(int month, int day, int year);
	void addYear(int yearCount);

	// ...
};
```

Now access to the private section is prohibited outside of Date's members.  The following code will now result in a compiler error:

```c++
Date linusTorvaldsBirthday;
linusTorvaldsBirthday.init(12, 28, 1969);


linusTorvaldsBirthday.day = 4000; // ERROR: cannot access private member
```

Alternatively we could change Date from a struct to a class.

```c++
class Date {
// implicitly private
	int day;
	int month;
	int year;
public:
	void init(int month, int day, int year);
	void addYear(int yearCount);

	// ...
}; 
```

The member variables (day, month, and year) are still private, because members are private by default within a class.

Even with a class we can (and probably should) be explicit about which parts are private:

```c++
class Date {
private: // explicitly private
	int day;
	int month;
	int year;
public:
	void init(int month, int day, int year);
	void addYear(int yearCount);

	// ...
};
```

A class interface is often used to express *invariant*, that is, things that cannot change.  For our Date class an invariant might be that it represents a legal date.  For a Url class it might be that it follows the rules of the [Uri specification](https://tools.ietf.org/html/rfc3986).

The public/private sections may appear in any order.  There can be any number of public/private sections within a type.  Different people have different styles with regard to preferred ordering and section counts.

Advantages of separating interface (public) from implementation (private):

- Illegal values/usage can be restricted.
- Implementation changes can be isolated to one type (callers need not change).
- Interface serves as documentation on how to use a type.
- Design can be improved.
- When making changes, finding all the places something private is used is easy.  If everything is public, it is harder (or sometimes impossible) to change.

Making members private can still be accessed via pointer tricks and other dirty mechanisms.  These mechanisms should be avoided  Makeing a member private does not mean it is somehow impossible for malicious code to access or more secure.  Writing such code is a dirty trick, and should be avoided.

```c++
Date date;
date.init(1, 1, 1970);

// DON'T DO THIS!!!
*reinterpret_cast<int*>(&date) = 4000; // change date.day to 4000
```

## class vs struct

In C++ class and struct can really be used interchangeably.  They do the same thing, except for different default rules for access specifiers (public/private).

That said a typical convention is to use structs only in the case of simple data records where all members are often public and in which few if any invariants are enforced.

Classes are typically used for more complex types that express invariants.

Classes are used most of the time in my experience.

## Constructors

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

### Constructor Overloads

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

### Default Constructors

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

#### Default 

Built-in types are considered to have default constructors, but they are not invoked

#### Generated Default Constructor

When there is no constructor written in a class the compiler will attempt to generate a default constructor that takes no arguments and simply calls the default constructor on each member variable.  If a constructor is specified, then the compiler will not generate a default constructor.

### explicit Constructors

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

When foo is called, it causes an implicit call to Date::Date(char const *), which creates an instance of Date.  That Date is then passed to foo.

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

## Destructors

We can also define special methods that can be used for cleanup.  Destructors are automatically called when the object is destroyed (when it goes out of scope).  Since destructors can be thought of as the complement of constructors, they use the same basic syntax except with the complement operator ~.

```c++
#include <iostream>

using namespace std;

class Array {
private:
    int * buffer;
public:
    explicit Array(int size);
    ~Array();
};

Array::Array(int size) {
    cout << "making buffer of size " << size << endl;
    buffer = new int[size];
}

Array::~Array() {
    delete [] buffer;
    cout << "buffer cleaned up" << endl;
}

int main(int argc, const char * argv[]) {
    cout << "Enter Count: ";
    
    int count;
    cin >> count;
    
    Array a(count);
    
    cout << "thank you and have a nice day!" << endl;
}
```

```
Enter Count: 10
making buffer of size 10
thank you and have a nice day!
buffer cleaned up
```

In this example we use the constructor to allocate memory from the free store (heap).  The destructor for Array automatically cleans up the memory when it goes out of scope (after the end of main).

Destructors never have arguments.

Destructors are also called automatically when heap allocated variables are deleted.  Consider the following:

```c++
Array * makeArray(int size) {
    return new Array(size);
}

int main(int argc, const char * argv[]) {
    cout << "Enter Count: ";
    
    int count;
    cin >> count;
    
    Array *a = makeArray(count);
    // ...
    delete a;
    
    cout << "thank you and have a nice day!" << endl;
}
```

```
Enter Count: 42
making buffer of size 42
buffer cleaned up
thank you and have a nice day!
```

This constructor/destructor pattern is known as *Resource Acquisition is Initialization* or *RAII* for short.  It can be used to manage more than just memory.  It can manage files, network connections, or any other kind of resource.  This is an extremely powerful mechanism of C++.  Once truly mastered it is one of the best parts of the language.

Note that properly using the types within the C++ standard library eliminates the need for manually calling new/delete.  We'll get to that later.

## Definition Order

Classes in C++ allow members to be written in any order.  Within a class we can use a member variable or method of that class before it is defined.  For example:

```c++
class Car {
private:
	int speed;
public:
	void setSpeed(int const s) {
		speed = s;
	}
};
```

Above the member variable speed is defined before it is used, but we could have also written it as follows:

```c++
class Car {
public:
	void setSpeed(int const s) {
		speed = s;
	}
private:
	int speed;
};
```

Some people prefer to list public parts of the class definition first so that readers will see the most relevant parts (the interface) first, but this is somewhat a matter of style.

## Static Members

We can specify members that are part of the class, but not specific to any instance.

### Static Member Variables

We can define member variables that are part of a class, but shared by all instances of that class by using the **static** keyword.

Take the following example:

```c++
#include <iostream>
using namespace std;

class Student {
private:
	static int _lastID;
	int _id;
public:
	explicit Student();

	int id();
};

Student::Student() {
	_id = ++_lastID;
}

int Student::id() {
	return _id;
}

int Student::_lastId = 0;

int main() {
	Student student1;
	Student student2;

	cout << "student 1: " << student1.id() << endl;
	cout << "student 2: " << student2.id() << endl;

	return 0;
}
```

With non-static (instance) member variables, each instance (e.g. student1, student2) its own member variables (e.g. _id).  With static member variables, the same variable is shared among all instances.  This means that if we make 100 instances of Student, there will be 100 Student::_id, but exactly 1 Student::_lastId.

### Static Methods

We can also define static methods.  A static method is not called from any instance of the object, but from the class.  static methods cannot access non-static (instance member variables.

We could define a static method to reset _lastId back to 0.

```c++
#include <iostream>
using namespace std;

class Student {
private:
	static int _lastId;
	int _id;
public:
	explicit Student();

	int id();
	static void reset();
};

Student::Student() {
	_id = ++_lastId;
}

int Student::id() {
	return _id;
}

void Student::reset() {
	//_id = 0;  // illegal reference to non-static member
	//id();     // illegal call of non-static member function

	_lastId = 0;

	Student student;
	student._id = 42; // legal (members can access private members)
}

int Student::_lastId = 0;

int main() {
	Student student1;
	Student student2;

	cout << "student 1: " << student1.id() << endl;
	cout << "student 2: " << student2.id() << endl;

	Student::reset();

	Student student3;
	Student student4;

	cout << "student 3: " << student1.id() << endl;
	cout << "student 4: " << student2.id() << endl;

	return 0;
}
```

Be careful with static variables.  Although their access and usage can be controlled (via a class interface and access modifiers) better than global variables, they are still essentially global variables.

## Self-Reference (this)

In non-static member functions we can access the instance from which the method was invoked via the **this** pointer.  The this pointer is read-only.  

One use of the this pointer is to disambiguate names.  Consider the following:

```c++
class Date {
private:
	int year;
public:
	Date(int year);
};

Date::Date(int year) {
	this->year = year; // assign the parameter to the member variable
}
```

Date's constructor has a parameter named year, but there is also a member variable named year.  Using the this pointer allows us to specify which one we meant.

Instead of using this to disambiguate, I typically use a underscore prefix convention to indicate member variables.  Consider:

```c++
class Date {
private:
	int _year;
public:
	Date(int year);
};

Date::Date(int year) {
	_year = year;
}
```

### Method Chaining

The this pointer can also be used to allow method chaining.  Consider the following:

```c++
class Date {
private:
	int _year;
	int _month;
public:
	Date& setYear(int year);
	Date& setMonth(int month);
};

Date& Date::setYear(int year) {
	_year = year;
	return *this;
}

Date& Date::setMonth(int month) {
	_month = month;
	return *this;
}

int main() {
	Date date;
	date.setYear(1992).setMonth(4); // method chaining

	return 0;
}
```

## Member Types

Classes can contain also contain types.  Member classes are frequently called *nested* classes.

Here Date defines a nested enum:

```c++
class Date {
public:
	enum CalendarType {
		Gregorian,
		Islamic,
		Hindu
	};
};

Date::CalendarType const type = Date::Gregorian;
```

Classes can also contain nested classes.  A nested class may access the private static variables of its containing class.

```c++
class A {
private:
	static int _foo;
	int _bar;
public:
	class B {
	public:
		int f() {
			return _foo; // legal
		}

		int b() {
			return _bar; // ERROR
		}
	};
};
```

Nested classes can access private non-static variables but only if they have a reference to the parent.  Consider:

```c++
class Parent {
private:
	int _a;
public:
	explicit Parent(int a);

	class Child {
	private:
		Parent * _parent;
	public:
		explicit Child(Parent *parent);
		int getA();
	};

	Child * makeChild();
};

Parent::Parent(int a) {
	_a = a;
}

Parent::Child::Child(Parent *parent) {
	_parent = parent;
}

int Parent::Child::getA() {
	return _parent->_a;
}

Parent::Child* Parent::makeChild() {
	return new Child(this);
}


int main() {
	Parent parent(42);

	Parent::Child *child = parent.makeChild();
	int a = child->getA();

	delete child;

	return 0;
}
```

<!--
**TODO:** file organization
-->