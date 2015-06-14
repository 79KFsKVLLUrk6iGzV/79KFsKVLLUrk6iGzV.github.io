# Containers

We can define classes that hold a set of values.  We typically call these classes *containers*.  The C++ standard library defines a large variety of containers (we'll see those later) and we can define our own.

In the section on [destructors](destructors.html) we already saw an example of a container.

Let's expand on our previous definition:

```c++
// Array.hpp
#pragma once

class Array {
private:
	int * _buffer;
	unsigned int _size;
public:
	explicit Array(unsigned int const size);
	~Array();

	int elementAt(unsigned int const index) const;
	int& elementAt(unsigned int const index);

	unsigned int size() const;
};
```

```c++
// Array.cpp
#include "Array.hpp"

Array::Array(unsigned int const size) : 
	_buffer(new int[size]),
	_size(size)
{
}

Array::~Array()
{
	delete [] _buffer;
}

int Array::elementAt(unsigned int const index) const
{
	// TODO: what if index is too big or small...we'll find out later
	return _buffer[index];
}

int& Array::elementAt(unsigned int const index)
{
	return _buffer[index];
}

unsigned int Array::size() const
{
	return _size;
}
```

Array is a container for integers (we'll see later how to make our containers generic so they can work with any type--but for now our containers only work with a single type).

We can get/set the values of elements within Array via the elementAt functions.  Notice that one of the methods is const (the getter) while the other is not (the setter).  Perhaps surprisingly the non-const method returns a element reference (int&), which allows the method call to be used where we would normally need a variable.  (Later we'll learn how to make accessing elements of our Array class more like accessing elements of built-in arrays.)

Array works a lot like the built-in array type, but it has some benefits.  With built in arrays we either need the size to be a compile-time constant or we need to allocate using new (and remember to delete[] when finished).  With our Array class, the user doesn't need to manually new/delete and the size doesn't need to be a compile-time constant.

For example:

```c++
unsigned int size;
cout << "enter a size: ";
cin >> size;

Array items(size);

for (unsigned int i = 0; i < size; ++i)
{
	cout << "enter a number: ";
	cin >> items.elementAt(i);
}
```

Unlike a dynamically allocated array, our Array class knows its own size.  This might be useful if we wanted to pass a reference to a function.  With a built-in dynamic array we would also need to pass the size, but with our Array class the size is built into the object.

Sometimes classes like Array are known as *resource handle*.  It acts as a handle to some resource (in this case dynamically allocated memory).  The object is responsible for making sure that the memory is properly freed.

Use of this container avoids *naked new/delete*.  This means that we avoid making direct new/delete outside very specific contexts, typically new in a constructor and delete in a destructor.  Code that does not use naked new/delete is much easier to be free of memory leaks.

Eventually we'll just use the built in types from the C++ Standard Library.  These types provide a large set of containers and other tools that make manual new/delete largely unnecessary.  For now however we should understand how these things work manually so we know how to correctly use the Standard Library classes and so we know how to build something ourselves if the Standard Library doesn't meet our needs.
