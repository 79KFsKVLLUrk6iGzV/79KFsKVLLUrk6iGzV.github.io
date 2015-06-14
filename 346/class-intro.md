# Class Basics

Let's start to look at how to use classes and objects.  At first we will consider concrete types.  These types are designed to model values.  This is called class-based or value-oriented programming.  This is really the basis of object-orientated programming.

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
