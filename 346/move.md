# Move

We now know how to write types that can safely be [copied](copying.html) (or at least types that forbid copying when it isn't safe), but we still have one last problem.  This problem is a lot more subtle than the disastrous problems we saw earlier.

## One Last Problem

Consider the following function:

```c++
Array makeArray()
{
	Array a(400000000);

	for (unsigned int i = 0; i < a.size(); ++i)
	{
		a.elementAt(i) = 42 + i;
	}

	return a;
}
```

If we call this function as follows:

```c++
auto const a = makeArray();
```

By the time this program reaches the beginning of the for loop it is using about 1.5 GB of RAM on my computer.  When it hits the return statement, the copy constructor is called.  Remember that returning a value causes it to be copied back to the caller, so it causes the copy constructor to be invoked (if we had disabled the copy constructor returning an Array wouldn't be allowed).  Let's look again at our copy constructor:

```c++
Array::Array(Array const &other) :
	_buffer(new int[other._size]), // allocate enough storage based on other
	_size(other._size)              // copy size
{
	copyElements(_buffer, other._buffer, _size);
}
```

What's going to happen when we hit that new?  Well on my computer it allocated another 1.5 GB of RAM (so our program is using about 3 GB of RAM at this point.

Also we're going to spend some (not a crazy amount for a person) time copying all those elements.

So everything does work here (at least if you compile the program for 64-bit), but we're doing a lot more work and allocating a lot more memory than we really should have to.

If we really think about that return statement we realize how silly this copy is.  When we copy a (the last line of makeArray), we're copying a value that we'll literally never be able to reference again.  Consider also Array is a resource handle.  It doesn't actually hold the values (they're stored in _buffer)--it just manages that resource.

What there was a way to simply steal the 'guts' (_buffer in the case of Array) of an object?  That is what if we could know that we're doing a copy from an object that can no longer be referenced (such as a in the last line of makeArray) and steal its _buffer object?  At that point we could avoid both the extra buffer allocation and the copy.

This operation is known as a *move*.  C++11 supports the ability to define the semantics of move in addition to just copy.  Move semantics are an important mechanism to improve efficiency.

## Move

Let's draw out a picture of what we're trying to accomplish:

![](move.png)

Looks great!!  No need to allocate additional memory, no need to copy the elements.  We simply copy the pointer and size to a new instance and zero out the previous instance.  Keep in mind that this operation is only safe if the value isn't going to be referenced anymore.  A great example of this is return values from functions.

## Back in the Bad Old Days

In the dark ages of pre-C++11 programs concern about the cost of copy operations resulted in many programmers not using return values for returning values.  This style of programming has many disadvantages, but it was still frequently done because of concern that excess copying would result in poor performance.

I am somewhat doubtful that excess copying was actually a frequent cause of poor performance, but at least there was a case to be made that returning 'large' values should be avoided due to cost.  You may still see this style of programming because C++11 is fairly new and it takes a long time for software to get updated and for people's habits to change.

In the bad old days a function like makeArray might have been written as follows:

```c++
void makeArray(Array &a)
{
	for (unsigned int i = 0; i < a.size(); ++i)
	{
		a.elementAt(i) = 42 + i;
	}
}

Array a(400000000);
makeArray(a);
```

There's a few problems with this style.  First of all it makes it pretty much impossible to use const variables (we need makeArray to be able to change a).  Also makeArray can't set the size of the buffer.  In addition the semantics are unclear.

If we just look at makeArray's signature it isn't obvious how the function should be used.  Is a an input to the function, and output from the function, or an input/output variable.  It could be any of those things from the signature.  Knowing how it works requires looking at the definition.  In this case the function is pretty simple, but in real world examples it might take a lot of time to understand the function or the source code might not be available.

## Syntax

Here's our Array class updated to support move:

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

	Array(Array const &other); // copy constructor
	Array& operator=(Array const &other); // assignment operator

	Array(Array &&other); // move constructor
	Array& operator=(Array &&other); // move assignment

	int elementAt(unsigned int const index) const;
	int& elementAt(unsigned int const index);

	unsigned int size() const;
};
```

```c++
// Array.cpp (truncated)

Array::Array(Array &&other) :
	_buffer(other._buffer),  // steal other's elements
	_size(other._size)
{
	other._buffer = nullptr;
	other._size = 0;
}

Array& Array::operator=(Array &&other)
{
	if (this != &other)
	{
		// clean up what we used to reference
		delete [] _buffer;
		_size = 0;

		// steal other's elements
		_buffer = other._buffer;
		_size = other._size;
		
		// "zero" out other
		other._buffer = nullptr;
		other._size = 0;
	}

	return *this;
}
```

Notice that like copy, supporting move requires defining both a move constructor and move assignment.  These operations are passed a non-const *rvalue reference*.  An rvalue represents a reference to a value that we cannot assign to, such as a temporary.  It is denoted by && (vs *lvalue reference* which is denoted by &).

When we define move operations the compiler will automatically bind to them instead of the copy operation when the value is a temporary value.  This is done via function overload.

Here's an example of a call that binds to the move constructor:

```c++
Array a = makeArray();
```

Because the move constructor is called when an object is being initialized with the value of a temp, we can just steal the temp's guts.  We also need to remember to null out the temp so that we prevent double deletes.

Here's an example of a call that binds to the move assignment:

```c++
Array a(0);
a = makeArray();
```

Since now we're moving a new value into an existing element we need to make sure that we free out the memory that was referenced by the object before stealing the other object's guts.

Also we need to check that we're not move assigning to ourself.  Let's see how that might happen:

```c++
Array a(0);
a = move(a);
```

move is a function defined in the C++ Standary Library that moves converts an lvalue into a rvalue (this is a destructive operation).  In this case we take value of a and move it into a, which calls Array's move assignment operator.  If we didn't have the if (this != &other) check then this operation would result in a dangling reference.

_buffer and other._buffer would have both been pointing to the same thing (since other and this would refer to the same object).  This means that the delete [] _buffer call would have freed the same buffer pointed to by other._buffer, so we we steal its pointer we're actually just stealing a pointer that points to memory we no longer own.

## Movable but not Copyable

It frequently makes sense to define class such that instances can be moved by they cannot be copied.  This works much like things in the real world.  Money can be moved between bank accounts, but it cannot (legally) be copied.

Similarly there are many types that might be movable but not copyable.  One example that we've used before is fstream.  fstream is resource handle for a file.

It makes sense that we could return an fstream from a function (move it from openLogFile back to the caller):

```c++
fstream openLogFile()
{
	return fstream("trace.log");
}

fstream fs1 = openLogFile();
fstream fs2 = fs1; // ERROR: copy not allowed
```

Why copying is not supported makes even more sense when you think about how fstream is implemented.  Typically an OS has an API for opening and closing files.  It might look something like this:

```c
// made up OS file open/close
void* openFile(char const *const filename);
void closeFile(void *handle);
```

These are a simplification.  Here are the actual [OpenFile](https://msdn.microsoft.com/en-us/library/windows/desktop/aa365430%28v=vs.85%29.aspx)/[CloseFile](https://msdn.microsoft.com/en-us/library/windows/desktop/ms724211(v=vs.85).aspx) functions in the Win32 API (the native API that allows Windows programs to interact with the operating system).

The openFile function returns a void\*.  This is basically just an address that represents an open file within the operating system kernel.  We don't really own this file.  When we're done with the file we need to tell the OS that we're done by calling closeFile and passing the void\* we were given when we called openFile.

With this knowledge we can now have a better understanding of fstream.  fstream is really just a class that wraps the OS file operations.  In its constructor it will call the native OS openFile routine and store the void pointer.  When the destructor runs it will free the file by calling closeFile.

It makes sense to copy things that we could consider values.  It makes sense that we could copy an integer for example.  It also makes sense that we could copy an Array, because it is a value that consists of an array of values.

fstream on the other hand does not represent a value--it represents a resource.  Even if we tried to copy the file, we wouldn't be copying the resource--we'd be making a new resource.  This doesn't actually work at all.  How would we be able to specify the name of this file via the copy constructor?

If we do want to pass an fstream to a function, we can't pass it by value because this requires a copy and fstream does not support copy.  We must instead pass the fstream by reference (or pointer).  For example:

```c++
void writeStuff(fstream &fs)
{
	// ...
}
```

## Rule of Five

In the previous section on copying, we learned about the Rule of Three.  Sometimes this is expanded for C++11 to be the Rule of Five.  The Rule of Five is a rule of thumb that claims that if a class needs to define any of the following it is likely that is should define them all:

- destructor
- copy constructor
- copy assignment operator
- move constructor
- move assignment operator