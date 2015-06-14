# Unit Testing

Unit testing is a way of making sure that our programs function as designed.  Think of how you might test a program:

1. Run the program with some test input
2. Check that the program outputs the expected result
3. Repeat for all test inputs

Now think about this process.  Is this an efficient way to find bugs in our programs?

How many times have you written part of you program, tested that it worked correctly, and then later found (perhaps when the assignment was returned) that changes after that point caused the earlier part to stop working?  This type of problem is know as a *regression*.  Our program went back to an earlier non-working state.

With unit testing we automate the process of manually running our programs with various inputs and manually verifying the output.  A program that runs through a series of tests automatically and reports the results.

## Examples

Let's start with a very simple example.  Our first examples are too basic for practical use, but they can serve as a conceptual introduction.

I have a function called inc.  It's purpose is to take the number that is passed as an input and return that number plus 1.

```c++
int inc(int const input) {
	return input + 1;
}
```

Now let's write a simple program that tests inc:

```c++
#include <iostream>
using namespace std;

int inc(int const input);

int main() {
	if (inc(0) != 1)      // test case 1
		return -1;
	if (inc(1) != 2)      // test case 2
		return -1;
	if (inc(-1) != 0)     // test case 3
		return -1;
	if (inc(42) != 43)    // test case 4
		return -1;
	if (inc(-42) != -41)  // test case 5
		return -1;

	cout << "all tests passed!" << endl;

	return 0;
}
```

I just defined 5 test cases.  The test cases just try different numbers and make sure that inc returns the expected value for each of the test inputs.

inc is obviously an extremely simple function.  It only has one branch.  If a function has more branches we'll generally expect to have more test cases.

If a function has too many branches than the function should be broken up into a set of smaller functions.

Let's do another example with a slightly more complex function:

```c++
int abs(int const input) {
	if (input >= 0)
		return input;
	else
		return -input;
}
```

Since this program has 2 branches, we'll probably want to write at least two tests.

```c++
#include <iostream>
using namespace std;

int abs(int const input);

int main() {
	if (abs(0) != 0)      // test case 1
		return -1;
	if (abs(1) != 1)      // test case 2
		return -1;
	if (abs(-1) != 1)     // test case 3
		return -1;

	cout << "all tests passed!" << endl;

	return 0;
}
```

## Benefits

What benefits could writing unit tests have?

- Find problems early
- Makes change easier
- Documents how code should be used
- Improves design
- Acts as a specification (executable specification)


## Scaling Up

In the previous examples we only had one function that we were trying to test.  For a more realistic program we would probably have various units (functions, classes, methods) that we wanted to test.

This leads to various questions: 

### *How do we organize all these tests?*

In our example we just had one function that we were testing.  We used main to run the test.  Since we only have one main function, we'll want to move our tests out of main.  Once we move the tests out of main, we need to somehow define which methods are tests and have a way of running them all.

### *How do we tell which tests are passing/failing?*

We would like our program to output a summary of which tests passed/failed.  We would also want to know if all the tests are passing or what percentage are passing.

### *How do we run specific tests?*

When a test starts failing, we want a way to run/debug a specific test.  This becomes especially important if we have a lot of tests.



## Unit Test Frameworks

We could probably design and build something to solve these problems, but there are already many unit testing frameworks available.  Instead of building something ourselves, let's just use something that's already been built for us. 

There are many C++ unit testing frameworks for us to choose from.  Here's a few that I know of:

- [CxxTest](http://cxxtest.com/)
- [googletest](https://code.google.com/p/googletest/)
- [Catch](https://github.com/philsquared/Catch)

## Catch

For this class we'll use Catch.  I choose Catch is that it is extremely easy to add to a project.

### Demo

```c++
// factorial.hpp

unsigned int factorial(unsigned int number);
```

```c++
// factorial.cpp

#include "factorial.hpp"

unsigned int factorial(unsigned int number) {
	return number <= 1 ? number : factorial(number - 1)*number;
}
```

We'll define something called a *static library* in that contains our factorial code.  A static library is basically just a collection of .o files.  You could give somebody else a static library (as well as your headers) and they could use your library.  A static library can be defined in CMake with the add_library command.

```cmake
# defines a static lib called factorial
add_library(factorial STATIC
  
  # files in factorial
  factorial.hpp
  factorial.cpp
)
```

Next we'll create an executable that will contain and run our unit tests.

First let's create a file called test-main.cpp to contain main for our tests.  Normally we would need to write main ourselves, but Catch actually writes main for us!

```c++
// test-main.cpp

#define CATCH_CONFIG_MAIN

// contains a definition of main
// (when CATCH_CONFIG_MAIN is defined)
#include "catch.hpp"
```

Next let's create a file that contains some tests.

```c++
// factorial-tests.cpp

// include catch
#include "catch.hpp"

// include what we're testing
#include "factorial.hpp"

TEST_CASE("Factorials are computed") {
	REQUIRE(factorial(1) == 1);
	REQUIRE(factorial(2) == 2);
	REQUIRE(factorial(3) == 6);
	REQUIRE(factorial(10) == 3628800);
}
```

Now we'll create our unit tests executable using CMake:

```cmake
# define an executable for our tests
add_executable(unit-tests
  catch.hpp   # this header is for Catch
  test-main.cpp
  factorial-tests.cpp
)
```

Also since our unit tests executable uses code defined in our factorial static library, we need to tell CMake to reference the factorial static library.  This is done with the *target_link_libraries* CMake command.

```cmake
# use our static library in the unit test exe
target_link_libraries(unit-tests
  factorial
)
```

The first argument is the name of the target (in our case the unit-tests executable) and the subsequent arguments are a list of libraries needed by the target.  In our case we just have one dependent library, factorial.

Finally we can build our program:

```bash
[]$ mkdir build
[]$ cd build/
[build]$ cmake ../  # only required the first time building this project
-- The C compiler identification is GNU 4.4.7
-- The CXX compiler identification is GNU 4.4.7
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Configuring done
-- Generating done
-- Build files have been written to: build
[build]$ cmake --build .
Scanning dependencies of target factorial
[ 33%] Building CXX object CMakeFiles/factorial.dir/factorial.cpp.o
Linking CXX static library libfactorial.a
[ 33%] Built target factorial
Scanning dependencies of target unit-tests
[ 66%] Building CXX object CMakeFiles/unit-tests.dir/test-main.cpp.o
[100%] Building CXX object CMakeFiles/unit-tests.dir/factorial-tests.cpp.o
Linking CXX executable unit-tests
[100%] Built target unit-tests
[build]$ 
```

Running the tests:

```bash
[]$ ./unit-tests
===============================================================================
All tests passed (4 assertions in 1 test case)
```

The unit-test executable supports many command line parameters.  See the [Catch command-line documentation ](https://github.com/philsquared/Catch/blob/master/docs/command-line.md) for more information.

### Assertions

Catch defines several macros that we can use to make *assertions*.  Assertions define the required conditions for our tests to pass.

#### TEST_CASE

Defines a test case.  Each test case is some some code (a function) that executes to verify some aspect of our program.

- The TEST_CASE macro takes a string as its first argument.  This string is the name of the test.  Catch requires that test names are unique.
- Each TEST_CASE defines a function that will be run when we run our tests.

#### REQUIRE

REQUIRE is used to define our expectations.

- The REQUIRE macro takes a boolean expression as an argument.  We pass an expression that we expect to be true.
- If the expression is false, then that test case fails.
- If the expression is false, then the rest of that TEST_CASE will not execute.

There is also a macro called CHECK that behaves like REQUIRE except that executation of the current test case will continue even if the assertion fails.

There are also REQUIRE_FALSE and CHECK_FALSE macros for when you expect something an expression to be false.

## Side Effects

So far our programs were pretty easy to test.  They were all basically mathematical functions.

Let's step back and think about the concept of a *function* vs a *subroutine*.  Frequently in programming we treat these terms as synonymous.  Many programming languages, C/C++ included, make no distinction.

### Functions

Functions come from the mathematical definition: 
> In mathematics, a function is a relation between a set of inputs and a set of permissible outputs with the property that each input is related to exactly one output. [Wikipedia](http://en.wikipedia.org/wiki/Function_(mathematics))

In programming a mathematical function is sometimes referred to as a *pure* function.  Pure functions are common in functional programming where the purity of a function may be enforced by the language.

Pure functions have several interesting properties:

- If the return value isn't used, the function call can be ignored
- If the function is called again with the same arguments it will always return the same value

*They have more interesting properties!!  But we'll leave those for another time.*

For testing these properties are really nice.  When testing a pure function we know that we will get the same output regardless of input.  The behavior of a pure function can be well understood by simply understanding that function.

### Subroutines

Subroutines (also known as procedure, function, routine, method, subprogram) is simply a callable unit.  A subroutine has an entry point and returns to its caller when it completes.  Pure functions are a type of subroutine, but subroutines are more general.

Consider the following:

```c++
#include <iostream>
using namespace std;

bool b;

int bozoInc(int const value)
{
	b = !b;
	
	return value + (b ? 1 : 42);
}

int main()
{
	cout << bozoInc(1) << endl;
	cout << bozoInc(2) << endl;
	cout << bozoInc(2) << endl;
	cout << bozoInc(1) << endl;

	return 0;
}
```

bozoInc is no longer a function in the mathematical sense.  It isn't a pure function.

bozoInc changes a non-local variable (b) as it executes.  This means that to test bozoInc we also need to control or understand the state of b.

Input/Output is another area of interest.  Consider the following subroutine:

```c++
#include <fstream>
using namespace std;

int age() {
	fstream fs("age.txt");

	int a;
	fs >> a;

	return a;
}

int main() {
	return age();
}
```

Now the age subroutine isn't a pure function because it depends on IO.  We can't really ascertain much about age(), even if it will work correctly without knowing more information, such as if a file named age exists, where it exists, what it contains, etc.

### Impact of Side-Effects on Testing

The better we can control and isolate side-effects the easier it will be to test our software.  The more things we need to setup and control the more difficult it becomes to write effective tests.  It also tends to make our tests run slower.

Writing testable software is one major motivation for OOP.  OOP gives us tools we can use to decouple units so they can be tested in isolation.  This will become more clear as we progress through the course.