# Copy

As we [saw earlier](default-copying.html), object in C++ can be copied.

How or whether class instances should be copied must be considered in the design of a class.

Remember that the default way an object is copied is by copying each of its members.  Hopefully we can frequently design our classes such that that default copying is sufficient, but sometimes it is not.

Let's look at a case where the default copying is disastrous.  When we wrote our [Array class](containers.html) previously we did not consider copying.  Unfortunately the default copying for Array isn't going to cut it.

Let's make an array and then copy it.

```c++
Array a(3);
	
for (unsigned int i = 0; i < a.size(); ++i)
{
	a.elementAt(i) = 42 + i;
}

Array b = a; // copy
```

First we make an Array instance called a.  Then we initialize a with 42, 43, 44.  Finally we copy a into another Array instance b.

Initially this seems to work.  But when the destructors for a and b run the program will most likely crash.

![](copy-layout.png)

Notice that both a and b have a pointer that points to the same buffer.  This is because the memberwise copy simply copied the members including the pointer address from a to b.

---

### Double Delete

The problem occurs when the destructor for Array runs.  When a and b go out of scope the Array destructor will be executed for each both a and b.  Unfortunately will cause the actual memory buffer pointed to by both a and b to be deleted twice.

This kind of problem is known as a *double delete.*  Forgetting to delete is a really bad thing (memory leak), but that doesn't mean deleting something more than once is a good thing.

We can also see a double delete if we tried to do something like this:

```c++
Foo *f = new Foo();
delete f; // remember calls ~Foo
//.. later
delete f; // calls destructor again 
```

Double delete is a serious problem because as soon as we free memory (via delete) the memory manager can give that memory out to somebody else.  Basically we don't own that memory any more.  When Foo's destructor runs for the second time its memory could have been overwritten by somebody else.

The worst thing about double delete is that *might* work depending on the specifics of what your program is doing.  This means that the program could work fine when you're testing it but still fail when your customer is using it.  A good compiler can sometimes add runtime checks (typically done only for debug builds) that can detect these problems when they occur at runtime (in general they cannot be detected at build time).

----

Let's look at how we can customize what it means for our type to be copied.

## Copy Constructor

Fixing this problem for involves defining a new type of constructor known as a *copy constructor*.  A copy constructor is how we override the default memberwise copy by defining the exact semantics of copy for our type.

Let's look at a copy constructor:

```c++
// Array.hpp

#pragma once

class Array {
private:
	int * _buffer;
	unsigned int _size;
public:
	explicit Array(unsigned int const size);
	Array(Array const &other); // copy constructor
	~Array();

	int elementAt(unsigned int const index) const;
	int& elementAt(unsigned int const index);

	unsigned int size() const;
};
```

```c++
// Array.cpp (truncated)
#include "Array.hpp"

Array::Array(Array const &other) :
	_buffer(new int[other._size]), // allocate enough storage based on other
	_size(other._size)              // copy size
{
	// copy the elements
	for (unsigned int i = 0; i < _size; ++i)
		_buffer[i] = other._buffer[i];
}
```

Notice that the copy constructor looks just like constructors we've seen previously, except it takes a const reference to another instance of itself.  Remember a class is like a blueprint that describes the structure that is common amoung all instances (all instances of Array have a _size and _buffer).  Each instance has particular values.  The copy constructor is a way of making a new instance of a class using the values from another already existing instance.

Now when we do:

```c++
Array b = a; // copy
```

our copy constructor gives each Array instance has its own copy of the buffer.

![](copy-layout-fixed.png)

## Copy Assignment

Unfortunately we still have another problem.  C++ objects can be created (or constructed) by copying an existing value (we just saw how to customize this with a copy constructor), but they can also be assigned.

During assignment an already created instance is given a new value.  This should be extremely familiar to us since we've been doing this since we started doing programming.

```c++
int a = 42; // copy initialization
int b = 43; // copy initialization

b = a; // copy assignment
```

Just like how we can do assignment for built-in types like int we can do assignment for our own custom types.  For example:

```c++
Array a(3);
	
for (unsigned int i = 0; i < a.size(); ++i)
{
	a.elementAt(i) = 42 + i;
}

Array b(2);

b = a; // copy assignment
```

As with copy constructors, every class is given a default assignment operator.  The default assignment operator simply assigns each member variable based on the value being assigned (again like the default copy constructor).  

In the example above when we assign the value of a to b this will cause the compiler to call the assignment operator.  Since we haven't explicitly defined how assignment should work for Array, the default compiler-generated assignment operator will be called.  The default assignment operator copies each member variable from a into b.  In other words a._buffer will be assigned to b._buffer and a._size will be assigned b._size.

As with the copy constructor, the default implementation is frequently exactly what we want and we should strive to write classes for which the default implementation is correct.  The default assignment operator behavior is completely disastrous for our Array.  It is even worst then the problem Array had with the default copy constructor.

If we draw out the problems become clear:

![](default-assignment-problems.png)

The problem is the default assignment behavior is incorrect for our array type.

As with copy, we fix the problem by giving a custom definition for assignment.  To do this we need to do something we haven't seen before--*operator overloading*.  The idea here is that we give a custom definition for what a built-in operator means for a user-defined type.  In this case we'll define what the assignment (aka =) operator means for Array.  Let's have a look:

```c++
// Array.hpp
#pragma once

class Array {
private:
	int * _buffer;
	unsigned int _size;
public:
	explicit Array(unsigned int const size);
	Array(Array const &other); // copy constructor
	~Array();

	Array& operator=(Array const &other); // assignment operator

	int elementAt(unsigned int const index) const;
	int& elementAt(unsigned int const index);

	unsigned int size() const;
};
```

```c++
// Array.cpp (truncated)
#include "Array.hpp"

namespace
{
	void copyElements(int *const destination, int const *const source, unsigned int const size)
	{
		for (unsigned int i = 0; i < size; ++i)
			destination[i] = source[i];
	}
}

Array::Array(Array const &other) :
	_buffer(new int[other._size]), // allocate enough storage based on other
	_size(other._size)             // copy size
{
	copyElements(_buffer, other._buffer, _size);
}

Array& Array::operator=(Array const &other)
{
	auto const copy = new int[other._size];
	copyElements(copy, other._buffer, other._size);

	delete [] _buffer;

	_buffer = copy;
	_size = other._size;

	return *this;
}
```

It might seem a little odd, but operator= is actually the name of the method.  As you can see the method performs the steps that would make sense for assignment.  Namely it takes the values from other and applies those values to itself.  The last 3 lines:

```c++
	_buffer = copy;
	_size = other._size;

	return *this;
```

are what the compiler provides by default.  In our customized version we do basically like what we did with the copy constructor where we copy the elements (instead of just copying the pointer) and then we free the old buffer before assigning the copy (to avoid leaking memory).

Both copy constructors and copy assignment operators should always take in their argument by const &.  This makes logical sense.  An assignment or copy operation should not alter the value of the value being copied.  This would be like a copy machine that altered or destroyed the paper being copied.

The assignment operator is required to return a reference to itself.  We've seen this before in the section on [self-reference](self-reference.html).  As you may remember we used this to make method chaining possible.  It is also done with the assignment operator to support method chaining.  You've seen this before, but perhaps it isn't obvious:

```c++
int a = 42;
int b = 43;
int c = 44;

a = b = c = 0; // method chaining
```

The assignment operator is required to return a reference to itself in support of the kind of chaining shown above (when applied to custom types).

## Copy Summary

A type correctly support copy if it supports both both copy construction and copy assignment.  These operations almost always come in pairs.  If you need a custom version of either, you almost certainly need a custom version of both.



## When is custom copy needed?

A custom copy definition is needed when the instances of that class should be copyable and the default compiler-generated copy definitions are incorrect.

One strong hint that the default copy implementation might not be correct when the type needs a destructor.  This was certainly true for Array.  The core problem is that _buffer is conceptually part of Array.  We know that the elements pointed to by _buffer should be thought of as part of Array.  If we want to copy an instance of Array, we should copy the values of _buffer.

We know this because we understand what the Array models and represents.  The compiler is clueless.  It only understands the grammar of the language.  We could define a type that had a pointer that was not logically part of that type's value.  In this situation we probably wouldn't want to free the memory referred to by the pointer in that type's destructor and we wouldn't want to provide a custom copy.

Another thing to keep in mind is that when we use types that already have correct copy semantics defined within another class, it is more likely that the default compiler-generated definitions of copy are correct.

This is a strong reason to use types defined within the C++ Standard Library.  These types will have the copy semantics properly implemented.  Later on in the semester we'll learn more about them.  For now it is important to learn how C++ works so we can more easily understand the types within the Standard Library.



## Rule of Three

The rule of three is a general rule of thumb for C++.  It states that if a class needs any of the following it probably needs all three:

- destructor
- copy constructor
- copy assignment operator

The rule makes sense when you realize that the compiler generates default implementations for these operators.  If the compiler-generated implementation for any of these operations is incorrect, then it is likely to be incorrect for all of them.


## Disabling Copy

Copy is a really important and powerful feature of C++.  The ability to provide custom definitions of copy operations allows things like automatic reference counting to be implemented libraries instead of requiring language features.

But this doesn't mean that we need to define how copy should work for every type.  Even when the default copy implementation is incorrect it doesn't mean that we need to *implement* copy semantics.  If we don't intend on actually copying a type, then we shouldn't busy ourselves implementing operations that aren't going to be used.

If the compiler-generated implementations are incorrect and we're not going to write an implementation (at least not for now), then we should prevent a future developer from accidentally calling an unsupported copy and encountering a nasty bug (like a double delete or memory leaks).  We can do this by disabling copy.

There are two ways to do this, an old way and a new way.  The new way is only supported in C++11 and should be used for new code.  Existing code might use the old pattern, so you should know about the old pattern in case you see it in code you're using.

Old Way:

```c++
#pragma once

class Array {
private:
	int * _buffer;
	unsigned int _size;

	Array(Array const &other); // copy constructor
	Array& operator=(Array const &other); // assignment operator
public:
	explicit Array(unsigned int const size);
	~Array();

	int elementAt(unsigned int const index) const;
	int& elementAt(unsigned int const index);

	unsigned int size() const;
};
```

Notice here that the copy constructor and assignment operator are listed in a private section.  If anybody attempts to call them they'll get an error about trying to use a private method.

Typically we'd wouldn't implement these methods at all in the .cpp file.  That way if we did accidentally call a copy operation from within our class we'd at least get a linker error instead of a run-time error.

New Way:

```c++
#pragma once

class Array {
private:
	int * _buffer;
	unsigned int _size;
public:
	explicit Array(unsigned int const size);
	~Array();

	Array(Array const &other) = delete; // copy constructor
	Array& operator=(Array const &other) = delete; // assignment operator

	int elementAt(unsigned int const index) const;
	int& elementAt(unsigned int const index);

	unsigned int size() const;
};
```

This syntax is only supported in C++11, but it is far better.  We simply say that the copy operations are deleted.  This means that we're not defining the operation and that the compiler should not generate a default implementation.  We don't define the methods in our .cpp file (if we did it would be an error).  If we attempt to call a copy operation (either internally or externally) we'll get a compiler error.

We still have one problem, but we'll leave that for the next section.