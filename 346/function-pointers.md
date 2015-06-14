# Function Pointers

## Background

When we write a function, the compiler will generate code for that function.  When our program runs, that function will exist somewhere in the computer's memory.

The vast majority of today's computers use the *Von Neumann architecture*.  One key characteristic of this architecture is that a single pool of memory (RAM) is used to store both data (e.g. variables) and instructions (compiled code). 

Each chunk of data within that pool has an address.  This means that everything in the pool can be addressed.  That means we can get the address of functions as well as data.

We've already seen how we can take an object in memory and obtain its address in memory:

```c++
int a = 42;
int *ptr = &a;
```

a stores an int whose value is 42.  ptr stores and address whose value is the location of a within the computer's memory.

This should all be familiar.  Now remember that the computer has one pool of memory.  Everything in that pool has an address.  The code generated for a function also exists in memory.  Therefore it must be true that functions themselves have addresses.

At the level of the machine, data and code are *homoiconic* (from Greek *homo* meaning same and *icon* meaning representation).  That is both data (variables) and instructions (functions) have the same form, bytes of data in RAM.  Some programming languages such as LISP support this concept at the level of the programming language, but C/C++ does not.

For reasons related to system design and machine architecture it is frequently not possible to modify the contents of memory containing code.  This commonly used as a technique prevent buffer overflow attacks (NX bit).

## C/C++

C and C++ support these concepts fairly directly with function pointers.  Since functions have addresses we can use a pointer to refer to a function.  Once we have a pointer to a function we can do all the things we could normally do with any other pointer (assign/initialize variables, pass to another function), but a pointer to a function lets us do one more thing--we can call the function at that address as we would call a normal function.

Function pointers do not provide a mechanism for modifying the code pointed to.

## Examples

In this example we'll initialize a variable that points to a function, call the function pointed to, then assign the pointer to a different function, call that function, and then finally assign the pointer to nullptr.

```c++
#include <iostream>
using namespace std;

void func1()
{
	cout << "func1" << endl;
}

void func2()
{
	cout << "func1" << endl;
}

int main()
{
	void (*fPtr)() = func1; // fPtr points to func1
	fPtr(); // call func1

	fPtr = func2; // fPtr points to func2
	fPtr(); // fPtr calls func2

	fPtr = nullptr; // fPtr points nothing

	return 0;
}
```

Probably the most strange thing in this example is the syntax for function pointers.  Notice the function pointer:

```c++
void (*fPtr)() = func1; // fPtr points to func1
```

This defines a variable named fPtr whose type is a pointer to a void function that takes no parameters.  fPtr is initialized to the address of the function func1.

This is a little different that what we're used to for variables.  Normally we first state the type and then we mention the name.  With function pointers we sort of state the type and the name all together.  The type basically surrounds the name.

Let's look at a few more examples:

```c++
int func3() { return 42; }

int main()
{
	int(*sally)() = func3;
	int returnValue = sally();

	return 0;
}
```

Here we define a variable sally whose type is a pointer to a function that takes no parameters and returns an int.  We can call the function sally points to by using the normal call syntax.  Notice that calling the function pointed to by sally returns a value.

We can also define a function pointer that accepts arguments:

```c++
#include <iostream>
using namespace std;

int add(int a, int b) { return a + b; }

int main()
{
	int(*mathOp)(int, int) = add;
	cout << mathOp(2, 2) << endl;;

	return 0;
}
```

We define a variable named mathOp that points to a function that accepts two parameters of type int and returns an int.


## Syntax

Every function pointer has a type that matches a function call signature.  You can only assign a function to a function pointer if that function's signature matches the type of the function pointer.  In other words the return type, argument count, and argument types for the function pointer must match any function you wish to assign to that pointer.

When calling a function pointer, the same rules that apply for normal function calls apply (argument count and types must match).  These rules apply to the type of the pointer.  For example, if the function pointer points to function that takes one int as a parameter and returns void, then when calling the function via the pointer you need to supply a single argument of type int.

While the syntax for function pointers isn't impossible to understand but it is a little much.  We frequently attempt to tame it slightly by using typedefs.  Typedefs aren't required to use function pointers (as seen earlier), but they can make our code much more readable.

Let's recreate the previous example using typedefs:

```c++
#include <iostream>
using namespace std;

int add(int a, int b) { return a + b; }

int main()
{
	typedef int(*MathOp)(int, int);

	MathOp mathOp = add;
	cout << mathOp(2, 2) << endl;;

	return 0;
}
```

Our typedef defines MathOp as a type that is a pointer to a function that accepts two ints as parameters and returns an int.  We use the typedef to define a variable named mathOp that we initialize to the address of add.  At this point the syntax for mathOp looks like other variables we're used to.

Normally when we take the address of something we use the address of (&) operator.  For functions we can't do anything other than get the address of the operator, so the address of operator is unneccessary.  It is not an error to use it, it is just unneccessary.

Let's see another example of using function pointers:

```c++
#include <iostream>
using namespace std;

int add(int a, int b) { return a + b; }
int sub(int a, int b) { return a - b; }
int mul(int a, int b) { return a * b; }

typedef int(*MathOp)(int, int);

MathOp getOp(char const ch)
{
	switch (ch)
	{
	case '+': return add;
	case '-': return sub;
	case '*': return mul;
	default: return nullptr;
	};
}

int main()
{
	int a, b;
	char operatorChar;

	cin >> a >> operatorChar >> b;

	auto const op = getOp(operatorChar);
	if (op == nullptr)
		cout << "not supported" << endl;
	else
		cout << "result: " << op(a, b) << endl;

	return 0;
}
```

Here we've made a very primative interpretor for mathematical expressions.  There's no reason we would have needed to use function pointers in this example, but the ability to do so gives us more design choices.  Let's see how this program might look without function pointers:

```c++
#include <iostream>
using namespace std;

int add(int a, int b) { return a + b; }
int sub(int a, int b) { return a - b; }
int mul(int a, int b) { return a * b; }

bool doMath(char const ch, int a, int b, int& result)
{
	switch (ch)
	{
	case '+': 
		result = add(a, b);
		return true;
	case '-':
		result = sub(a, b);
		return true;
	case '*':
		result = mul(a, b);
		return true;
	default:
		return false;
	};
}

int main()
{
	int a, b;
	char operatorChar;

	cin >> a >> operatorChar >> b;

	int result;
	if (doMath(operatorChar, a, b, result))
		cout << "result: " << result << endl;
	else
		cout << "not supported" << endl;

	return 0;
}
```

This version also works just fine.  But notice how the design is slightly different.  With function pointers we were able to isolate the mapping between a char that represents a mathematical operator to its corresponding function to a function dedicated only to that task.  In the version without function pointers the mapping and performing the operation must both be done in the same function.

Which version is better?  These examples are so simple it is hard to say, although I do find the first version to have a certain elegance that second version lacks.  The key take away is that function pointers allow us to design programs differently and allow for decoupling of concerns (in our case the mapping and the calling).

## Callbacks

We might write a function that takes another function as a parameter.  The function argument will be called periodically to return values.  We can basically think of this as a function that is able to deliver a set of results incrementally as its work progresses.

As a simple example, consider the writing a function to compute Fibonacci numbers.  Since we want our function to be reusable in a variety of contexts we cannot output the numbers to the console as they are computed.  We want to keep our function general so it could be used in a variety of programs.  What if we wanted to use our function in a GUI application?  Or a web page?  Or for some other computation?  Writing to the console isn't a very general solution.

So what should the function return?  We could return something like a vector.  This could work pretty well, but if we wanted to do a large number of Fibonacci numbers this could get inefficient.  Wouldn't it be nice if we could just have our function return the Fibonacci numbers as they are calculated?

Let's look at an example:

```c++
#include <iostream>
using namespace std;

typedef bool(*fibonacci_callback)(int, int);

void fibonacci(fibonacci_callback callback)
{
	int index = 0;

	if (!callback(index++, 0))
		return;
	if (!callback(index++, 1))
		return;

	int prev2 = 0;
	int prev1 = 1;

	while (true)
	{
		int current = prev2 + prev1;

		if (!callback(index++, current))
			return;

		prev2 = prev1;
		prev1 = current;
	}
}

bool print(int index, int number)
{
	cout << number << endl;
	return index < 10;
}

int main()
{
	fibonacci(print);

	return 0;
}
```

Now our fibonacci function is generic.  The function itself doesn't write to the console, it simply delivers its results to the callback.  It doesn't need to store the entire sequence in memory before it starts delivering results, so we could list millions of numbers without wasting memory or causing an initial delay.

Let's see another example:

```c++
#include <iostream>
using namespace std;

typedef bool(*fibonacci_callback)(int, int);

void fibonacci(fibonacci_callback callback);

int accumulator;
bool sum(int index, int number)
{
	accumulator += number;
	return index < 10;
}

int main()
{
	accumulator = 0;
	fibonacci(sum);

	cout << accumulator << endl;

	return 0;
}
```

In this example we use the same fibonacci function to add up Fibonacci numbers.  To keep track of the running total we use a global variable.

This works ok, but as we've learned already global variables aren't very desirable from a design perspective.  How could we modify this program so that we don't need to use global variables?  The problem is that we need some data or context information that exists longer than sum.  Global variables work, but then we're using global variables.  We could pass a variable to fibonacci, but we can't pass it anything specific.  Typically this is handled with a void pointer:

```c++
#include <iostream>
using namespace std;

typedef bool(*fibonacci_callback)(int, int, void *);

void fibonacci(fibonacci_callback callback, void * context)
{
	int index = 0;

	if (!callback(index++, 0, context))
		return;
	if (!callback(index++, 1, context))
		return;

	int prev2 = 0;
	int prev1 = 1;

	while (true)
	{
		int current = prev2 + prev1;

		if (!callback(index++, current, context))
			return;

		prev2 = prev1;
		prev1 = current;
	}
}

bool sum(int index, int number, void * context)
{
	*(reinterpret_cast<int *>(context)) += number;
	return index < 10;
}

int main()
{
	int accumulator = 0;
	fibonacci(sum, &accumulator);

	cout << accumulator << endl;

	return 0;
}
```

A void pointer can point to anything.  This is really important.  If we wouldn't have passed a pointer to an int, than fibonacci would have worked fine in the case of sum, but it wouldn't work for other cases.  What if we wanted to also provide the 10 as a parameter to make our sum function more generic?

```c++
#include <iostream>
using namespace std;

typedef bool(*fibonacci_callback)(int, int, void *);

void fibonacci(fibonacci_callback callback, void * context)
{
	int index = 0;

	if (!callback(index++, 0, context))
		return;
	if (!callback(index++, 1, context))
		return;

	int prev2 = 0;
	int prev1 = 1;

	while (true)
	{
		int current = prev2 + prev1;

		if (!callback(index++, current, context))
			return;

		prev2 = prev1;
		prev1 = current;
	}
}

struct sum_context
{
	int accumulator;
	int maxIndex;
};

bool sum(int index, int number, void * context)
{
	sum_context *ctx = reinterpret_cast<sum_context*>(context);

	ctx->accumulator += number;
	return index < ctx->maxIndex;
}

int main()
{
	sum_context ctx;
	ctx.accumulator = 0;
	ctx.maxIndex = 10;

	fibonacci(sum, &ctx);

	cout << ctx.accumulator << endl;

	return 0;
}
```

## Indirection

When we call a function directly, we simply use the function's name in the source code.  When we call a function using a function pointer we're adding a level of indirection.  Instead of referring to the function by its name, we're referring to it via its address.  This allows the caller to not have to be specific about the exact function it is calling.  It simply is calling some function that meets a certain function signature.

It is often stated that every problem in computer science can be solved by another level of indirection, except of course the problem of too much indirection.

Indirection is a topic of huge interest with regard to OOP as we'll see soon.