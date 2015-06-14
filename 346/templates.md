---
title: Templates
---
# Templates

Templates are a huge topic.  We're not going to cover templates in great detail, but hopefully this should be a good introduction so you're able to use most templates written by others and write simple templates of your own.

## Motivation

In the previous section on [containers](containers.html) we learned about objects that are meant to hold a set of objects.  In that section we defined a class named Array that could store a set of ints.

Consider the following program:

```c++
#include <iostream>
using namespace std;

class Array {
private:
    int * _buffer;
    unsigned int _size;
public:
    explicit Array(unsigned int const size) : 
        _buffer(new int[size]),
        _size(size)
    {
    }
    
    ~Array() {
        delete [] _buffer;
    }

    int elementAt(unsigned int const index) const {
        return _buffer[index];
    }
    
    void setElementAt(unsigned int const index, int element) {
        _buffer[index] = element;
    }

    unsigned int size() const {
        return _size;
    }
};


int main() {
    Array a(2);
    a.setElementAt(0, 1);
    a.setElementAt(1, 2);
    
    for (int i = 0; i < a.size(); ++i)
    {
        cout << a.elementAt(i) << endl;
    }
    
    return 0;
}
```

Now suppose that we want to expand our program to also include an Array that works with doubles:

```c++
#include <iostream>
using namespace std;

class ArrayOfInt {
private:
    int * _buffer;
    unsigned int _size;
public:
    explicit ArrayOfInt(unsigned int const size) :
        _buffer(new int[size]),
        _size(size)
    {
    }
    
    ~ArrayOfInt() {
        delete [] _buffer;
    }

    int elementAt(unsigned int const index) const {
        return _buffer[index];
    }
    
    void setElementAt(unsigned int const index, int element) {
        _buffer[index] = element;
    }

    unsigned int size() const {
        return _size;
    }
};

class ArrayOfDouble {
private:
    double * _buffer;
    unsigned int _size;
public:
    explicit ArrayOfDouble(unsigned int const size) :
        _buffer(new double[size]),
        _size(size)
    {
    }
    
    ~ArrayOfDouble() {
        delete [] _buffer;
    }

    double elementAt(unsigned int const index) const {
        return _buffer[index];
    }
    
    void setElementAt(unsigned int const index, double element) {
        _buffer[index] = element;
    }

    unsigned int size() const {
        return _size;
    }
};

int main() {
    ArrayOfInt a(2);
    a.setElementAt(0, 1);
    a.setElementAt(1, 2);
    
    for (int i = 0; i < a.size(); ++i)
    {
        cout << a.elementAt(i) << endl;
    }
    
    
    ArrayOfDouble b(3);
    b.setElementAt(0, 4.2);
    b.setElementAt(1, 4.5);
    b.setElementAt(2, 4.9);
    
    for (int i = 0; i < b.size(); ++i)
    {
        cout << b.elementAt(i) << endl;
    }
    
    return 0;
}
```

Notice any problems?  This works, but there is a lot of duplication between ArrayOfInt and ArrayOfDouble.  Wouldn't it be nice if we could define a template from which a set of similar classes or functions could be defined?

## Macros

We could solve this problem using macros, but macros introduce their own set of problems.

- Macros frequently screw up programming tools (code completion, debuggers, renaming, refactoring)
- Macros know little about C++ rules or syntax.  They don't respect scoping rules for example.
- Macros often lead to obscure error messages.
- Macros create a private language that will be more difficult for other programmers to read.

Macros are pretty unavoidable in C code, but C++ usually has a better alternative.  C++ programmers should strive to avoid when possible.

## Templates

A *template* is a function or class that we parameterize with a set of types or values.  Templates allow us to express general concepts and then generate specific types by applying types or values as parameters.  For example it would allow us to express the general concept of an Array (which may store elements of any type), and then generate specific classes for an Array containing ints or an Array containing doubles.

Let's see an example:

```c++
#include <iostream>
using namespace std;

template <typename T>
class Array {
private:
    T * _buffer;
    unsigned int _size;
public:
    explicit Array(unsigned int const size) :
        _buffer(new T[size]),
        _size(size)
    {
    }
    
    ~Array() {
        delete [] _buffer;
    }

    T elementAt(unsigned int const index) const {
        return _buffer[index];
    }
    
    void setElementAt(unsigned int const index, T element) {
        _buffer[index] = element;
    }

    unsigned int size() const {
        return _size;
    }
};

int main() {
    Array<int> a(2);
    a.setElementAt(0, 1);
    a.setElementAt(1, 2);
    
    for (int i = 0; i < a.size(); ++i)
    {
        cout << a.elementAt(i) << endl;
    }
    
    
    Array<double> b(3);
    b.setElementAt(0, 4.2);
    b.setElementAt(1, 4.5);
    b.setElementAt(2, 4.9);
    
    for (int i = 0; i < b.size(); ++i)
    {
        cout << b.elementAt(i) << endl;
    }
    
    return 0;
}
```

Let's break down what we just did.  The following line:

```c++
template <typename T>
```

Makes T a type parameter of the declaration/definition that follows (in this class the class Array).  Note that T is simply an identifier that refers to a type.  We could have named it something else like TElement.  The names don't need to start with T, but that is a common convention to refer to type parameters.

You could have more than one type parameter, for example:

```c++
template <typename T1, typename T2>
```

You can also use the keyword class instead of typename.  For example:

```c++
template <class T>
```

In this context class and typename are completely interchangeable.  Both refer to a type.  Even if you say class it doesn't need to be a class, it could be a struct or primitive type.  class is less to type that typename, but typename is more clear to readers.  I prefer to use typename, but you should know both so you can read code written by others.

After we defined Array as a template, we can use the template parameters anywhere within the class definition as though they were types.  In Array we use the template parameter T in place of where we used int or double previously.

## Template Instantiation

So how do templates actually work?  The first thing to understand about templates is that they are purely a compile-time mechanism.  Unlike virtual methods which are implemented using both compile-time and run-time mechanisms, templates are completely compile-time.

In our program we instantiated the Array template twice.  Remember we had:

```c++
Array<int> a(2);
Array<double> b(3);
```

The compiler will generate code for Array<int> and Array<double>.  This generation happens at compile-time.  We never see the generated code, but it is basically identical to what we had written manually (ArrayOfInt, ArrayOfDouble).

The compiler will only generate a single class for each unique set of template parameters.  In our example we only made one instance of Array<int>, but if we had made another the compiler would only generate the code for Array<int> once.

Since this generation is a purely compile-time mechanism, it doesn't matter if the generated code is actually needed at run-time or not.  Consider the following example:

```c++
int main() {
    string answer;
    cout << "int or double: ";
    cin >> answer;

    if (answer == "int") {
        Array<int> a(2);
        a.setElementAt(0, 1);
        a.setElementAt(1, 2);
        
        for (int i = 0; i < a.size(); ++i)
        {
            cout << a.elementAt(i) << endl;
        }
    }
    else if (answer == "double") {
        Array<double> b(3);
        b.setElementAt(0, 4.2);
        b.setElementAt(1, 4.5);
        b.setElementAt(2, 4.9);
        
        for (int i = 0; i < b.size(); ++i)
        {
            cout << b.elementAt(i) << endl;
        }
    }
    
    return 0;
}
```

In this program we're going to use either Array<int> or Array<double> but not both.  Still code will be generated for both.  This is because at compile-time there is no way to determine which option the user will pick.  There needs to be code in place for both options.

## Template Functions

We can also define function templates.  Like class templates function templates allow us to parameterize the types that we wish to use.  Here's a simple example:

```c++
template <typename T>
T const & max (T const &a, T const &b) {
  return (a < b) ? b : a;
}
```

This function template will find the larger of a and b.  Let's see some examples of calling max:

```c++
max<int>(42, 43);
max<double>(4.3, 4.9);
max<char const *const>("bbb", "aaa");
max<string>("bbb", "aaa");
```

A function template's template arguments can be deduced automatically by from the types of the function arguments.  We can, for example, do any of the following: 

```c++
max(42, 43);
max(4.3, 4.9);
max("bbb", "aaa");
max(string("bbb"), string("aaa"));
```

Notice that we didn't explicitly specify the template argument types.  Since the function arguments used all the template argument types, the compiler was able to infer the template types based on the function arguments.  Note that the compiler can only infer types with function templates, not class templates.

We've seen this kind of type inference before: the auto keyword.  auto is new with C++ 11, but type inference for template functions has been around for a while in C++.

If not all the types can be inferred based on the function arguments then the template arguments need to be explicitly specified.  We've already seen an example of this:

```c++
max<string>("bbb", "aaa");
```

Here the type can be inferred, but the inferred type is char const *const, and the < operation for pointers is probably not what we want.  If we want to find the lexicographical maximum, we need to use a type that supports < operations, such as the built in string type.

If only some of the arguments can be inferred from the arguments, then we can specific only some of the template arguments.  All the template arguments that can be inferred need to be at the end of the template argument list.  For example:

```c++
template <typename TElement, typename TContainer>
TElement front(TContainer const &container)
{
    return container.elementAt(0);
}

int main() {
    Array<int> a(2);
    a.setElementAt(0, 42);
    a.setElementAt(1, 44);
    
    cout << front<int>(a) << endl;
    
    return 0;
}
```
Notice that TElement needed to be explicitly provided in the template arguments (as int), but TContainer can be inferred by the argument to be Array<int>.  I purposefully put TContainer as the second template argument.  This was so when front is called TContainer would typically not need to be specified.  Remember that if we don't specify need to specify some template arguments and have others inferred, we need all the inferred types to be at the end.

## Duck Typing

When we write a template, we'll frequently want to call operations on objects passed to the template.  We've already seen many examples of this.  Consider the following:

```c++
template <typename T>
T const & max (T const &a, T const &b) {
  return (a < b) ? b : a;
}
```

Consider that we compare a and b using the < operator. So we could do this:

```c++
max(42, 43);
```

But what if we try to do the following?

```c++
struct Foo
{
};

int main() {
    Foo a;
    Foo b;
    
    max(a, b);
    
    return 0;
}
```

In the first case our program will work without any problem because the types that we use with our template (int) support the < operator, but in the second case it won't compile.  The problem is that while primitive types like int support the < operator, user-defined types like Foo only support operator < if explicitly defined using operator overloading.  Since our template uses the operator < on the operands any template arguments supplied to this template will need to support the < operator.

Keep in mind that this is entirely a compile-time operation.  The compiler will determine if there is any errors, and the actual call binding (the binding of the < operator to the appropriate method given the template argument) all happens at compile time.

The operations we use are not limited to operators.  We've already seen an example of this also.  Consider:

```c++
template <typename TElement, typename TContainer>
TElement front(TContainer const &container)
{
    return container.elementAt(0);
}
```

Here we need whatever is type we use for TContainer to have a method const method named elementAt that takes an integer as the first argument and returns the type specified by TReturn.  If we attempt to call front with an object that does not meet these requirements, our program will not build.

When templates are instantiated, the compiler will check that the arguments support the operations that could potentially be used on that type within the template.  This typing mechanism is sometimes known as **duck typing**.   That is "if it walks like a duck and quacks like a duck then it's a duck".

## Errors

The duck typing mechanism used by templates has a deep effect on the errors that can occur with templates.  Normally when we write a function or class the compiler will check for any errors within that code while that code is compiled.  If we try to call a method that doesn't exist for example the compiler will report that error at that spot in the function.

Templates are a little different.  When the compiler examines the template itself there are pretty severe limits on what errors can be reported for operations performed on template argument types.  Consider the following:

```c++
template <typename T>
T add(T const &a, T const &b) {
    return a + b;
}
```

A basic error like a missing semicolon can be detected when the template is examined by the compiler.  Consider the parameters a and b however.  The template expects that whatever type is used for a and b should support an + operator that returns that same type.

So we can do something like:

```c++
add(2, 2);
add(2.0, 2.0);
```

But something like the this:

```c++
Foo a, b;
add(a, b);
```

will not work.  Foo doesn't support the + operator.

When we instantiate the template with specific types the compiler will have more information about the operations that are allowed on the type, so it will be able to produce more errors.

Unfortunately errors caused by using types that are not supported by the template will be reported as errors within the template itself.  Consider the following:

```c++
struct Foo {};

template <typename T>
T add(T const &a, T const &b) {
    return a + b;  // ERROR: invalid operands
}

int main() {
    Foo a, b;
    add(a, b);
    
    return 0;
}
```

The problem is really that we're using a type that doesn't support the + operator, but the compiler will report that error inside the add template.

Usually if you look at the output log from the compiler carefully you'll be able to understand the actual cause of the problem.  For example, here's the clang compiler error message for the previous program:

```
error: invalid operands to binary expression ('const Foo' and 'const Foo')
    return a + b;
           ~ ^ ~
note: in instantiation of function template specialization 'add<Foo>' requested here
    add(a, b);
    ^
```

The first line is saying that there is a problem in the template because Foo doesn't support the + operator.  The second line note is saying that the compiler was trying to instantiate add with Foo as the template argument.  Often times when working with templates these notes are a lot more important than the actual location of the error.

## Non-Type Template Arguments

Templates can be used for non-type arguments as well.  Consider the following:

```c++
template <typename T, size_t size>
struct Buffer {
T Elements[size];
};
```

Remember that a stack allocated array needs to have a size that is known at compile-time.  By making the array size a template parameter it is known at compile-time.  This is because values that we supply as template arguments need to be compile time constants.  Consider how we use Buffer:

```c++
Buffer<int, 2> buffer;  // OK: 2 is a compile-time constant
```

```c++
int a;
cin >> a;
Buffer<int, a> buffer; // ERROR: non-type template argument is not a constant expression
```

## Libraries

The way that errors are reported from templates is especially unfortunate given that many times the templates a programmer uses will written in a 3rd party library.  When using template libraries a programmer will often find that the compiler reports errors located within the library itself.

Usually the problem is actually in how the template is being used and not in the library (especially when it comes to widely used libraries like the STL), but it can often be extremely difficult to decipher the messages reported when using templates.

Here's an example using the C++ STL:

```c++
vector<int const> elements;
elements.push_back(1);
```

If we attempt to compile a program containing that code, we might see the following compiler error:

```
In file included from main.cpp:1:
In file included from /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/../include/c++/v1/iostream:38:
In file included from /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/../include/c++/v1/ios:216:
In file included from /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/../include/c++/v1/__locale:15:
In file included from /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/../include/c++/v1/string:439:
In file included from /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/../include/c++/v1/algorithm:627:
/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/../include/c++/v1/memory:1550:27: error: cannot initialize a parameter of type 'void *' with an lvalue of type 'const int *'
            _VSTD::memcpy(__end2, __begin1, _Np * sizeof(_Tp));
                          ^~~~~~
In file included from main.cpp:3:
/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/../include/c++/v1/vector:839:21: note: in instantiation of function template specialization 'std::__1::allocator_traits<std::__1::allocator<const int> >::__construct_backward<const int>' requested here
    __alloc_traits::__construct_backward(this->__alloc(), this->__begin_, this->__end_, __v.__begin_);
                    ^
/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/../include/c++/v1/vector:1528:5: note: in instantiation of member function 'std::__1::vector<const int, std::__1::allocator<const int> >::__swap_out_circular_buffer' requested here
    __swap_out_circular_buffer(__v);
    ^
/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/../include/c++/v1/vector:1561:9: note: in instantiation of function template specialization 'std::__1::vector<const int, std::__1::allocator<const int> >::__push_back_slow_path<const int>' requested here
        __push_back_slow_path(_VSTD::move(__x));
        ^
/Users/Kevin/Dropbox/demo/demo/main.cpp:58:10: note: in instantiation of member function 'std::__1::vector<const int, std::__1::allocator<const int> >::push_back' requested here
elements.push_back(1);
         ^
In file included from main.cpp:1:
In file included from /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/../include/c++/v1/iostream:38:
In file included from /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/../include/c++/v1/ios:216:
In file included from /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/../include/c++/v1/__locale:15:
In file included from /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/../include/c++/v1/string:436:
In file included from /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/../include/c++/v1/cstring:61:
/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.10.sdk/usr/include/string.h:72:20: note: passing argument to parameter here
void    *memcpy(void *, const void *, size_t);
                      ^
1 error generated.
```

This giant mess of errors is the result of attempting to use const int as the type passed to the vector template.  The problem is that vector requires that the element type support assignment, and const ints can't be assigned.  The fact that templates can't express these kinds of requirements at their interfaces means that we end up getting error messages that are exceptionally detailed and not particularly helpful.

I know this was a huge area of frustration for myself when I first started using STL.  Since I didn't fully templates, I was really confused by all the error messages being reported within STL itself.  I thought that the library must have been really poorly written if I was getting compiler errors with the library.  Actually it couldn't be further from the truth--STL is extremely well written.  The actual problem was my incorrect use of the STL and my misunderstanding of templates, but this problem was worsened by the poor error reporting of templates.

## Concepts

The core problem with template error reporting is there is no way to express the requirements of a template argument.  The only way to determine the requirements of a template argument is to examine the actual template.  This means that the compiler can't determine an error until it is actually trying to compile the code that it generated from a specific instantiation of a template.  At that point the error message provided will be in the template code itself, and especially when that template code was written by a different programmer, it might be difficult to understand.

Basically the problem is that checking the template arguments is done too late in the compilation process to be overly useful to the programmer using the template.  Since it is done so late the error messages are frequently too low-level.

The C++ Standards Committee is well aware of these issues and has been working for quite some time to fix it.  There is a proposal for a C++ feature called concepts which would allow template authors to specify the template argument requirements as part of the template interface.  This would allow compilers to report much more user-friendly error messages when templates are used with incorrect template arguments.

Unfortunately C++ Concepts is still a work in progress and isn't a part of any current compilers.  At one point is was planned to be included in the specification for C++11, but the committee decided that it needed more time to mature before being added to the C++ Standard.

## Performance

Since templates are a compile-time mechanism, the performance is very good.  It should be assumed that templates will have the same performance as if we would manually copy the template manually for each template instantiation.

Heavy use of templates can lead to larger executable sizes, do to the amount of code generated by the templates.  This isn't usually a major concern, but if overall executable size is a concern then you might want to be careful with templates.

It is possible that the larger code size may cause some performance degradation, due to less optimal use of CPU caches.  This is a pretty extreme edge case.  I've never actually encountered a performance problem related to templates, but I have seen larger executables due to templates.

## File Organization

With a non-templated function/class we can put the declaration in the header file and the definition in a source file.  This is highly desirable because it allows our functions to be compiled once.  Anybody that wants to use the function from a different source file simply includes the header file that contains the declarations of the functions.  

Given the declarations the compiler is able to make sure that the function is called correctly.  The compiler doesn't actually know the address of the function.  The function might exist in a different translation unit.  Until link-time the address (technically the address offset) is simply left as a TODO for the linker to fill in.

At link-time the linker will look at all the functions and all the call site TODOs.  It will replace the TODOs with the offsets of the actual functions.  When the load is loaded into memory to run the address offsets will be translated by the loader to actual memory addresses where the functions exist in memory.

This should all be review from the previous sections on linking.  Templates are a little different.

When a template is instantiated the compiler needs the entire template definition.  The compiler is responsible for taking the template definition and generating a function/class with the given template arguments.  In order to do this the compiler needs the entire template definition.  Simply having the declarations is not enough.

The template is not a class/function.  It is just a template.  No code is generated for a template until the template is instantiated.

Template generation happens at compile-time, not link-time.  This means that for templates we cannot use the same put-the-declarations-in-the-headers-and-definitions-in-the-source approach.  That approach would have required the linker to be responsible for template instantiation.  This is way beyond the job that the linker would normally do.  Only one C++ compiler ever attempted to implement this feature, and it has subsequently been removed from the language.  It may eventually return, but for now it is not possible.

What this all means is that we typically place the entire template definition in the headers.  A templated class for example, will typically not have a .cpp file, only a .hpp.  Wherever the template is included the compiler will have the full definition.  We can either define the class methods inline or externally, but it all needs to be in the same header.

Unfortunately this also means that templates can have a huge negative impact on compilation speeds.


## Templates vs Interfaces (abstract classes)

When we looked at interfaces we saw a way of defining generic software based on abstract classes.  The interface defined the requirements of objects that were passed to a function.  We could then write generic functions against those types.

Considering templates, they provide a similiar although subtly different mechanism.  Generally abstract classes will be much easier to read/write and have faster compile times.  Templates will typically have better run-time performance, but they are harder to read/write and will be slower to compile.

Abstract classes are also more flexible.  They allow components to be swapped at run-time where with templates we can only swap components at build-time.

If we define an interface using templates then when we pass around components that implement that interface around, all of that code will need to use templates as well.  This can lead to an explosion of templates.

Frequently libraries that have a heavy focus on maximizing run-time performance will make heavy use of templates.  You'll find this commonly throughout the STL and many other C++ libraries.

I mainly use templates for collections and writing generic algorithms.  I tend to avoid templates when I could easily use an abstract interface instead.  Although templates have many powerful uses--and I do use them quite frequently--I tend to prefer abstract classes for their better build speed, understandability, and simpler error messages.

