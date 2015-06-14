# C/C++ Compilation Model

When compiling C/C++ programs we often feed a list of .c/.cpp/.cc files to g++ and have it produce an output program.

Example:
```
[]$ g++ foo.cpp main.cpp
```

The preceding command will create an executable (a.out) that we can run:
```
[]$ ./a.out
hello world
```

There are actually many more steps in the build process.  For the purposes of this class we will examine 4 stages (preprocessor -> compiler -> assembler -> linker), but there are actually more stages.  By default, when we invoke gcc, as in the preceding example, all of these steps are combined into a single process, but we can also run these steps separately by supplying gcc with the correct command line flags.

While this single-step build works ok for small programs, it does not scale well.  Whenever any part of the program changes, the entire program needs to be re-built.

Let's examine the compilation process in a little more detail, given the following source files:

```c++
// foo.hpp

int foo();
```

```c++
// foo.cpp

#include "foo.hpp"

int foo()
{
        return 42;
}
```

```c++
// main.cpp

#include "foo.hpp"

int main()
{
        return foo();
}
```

## Preprocessor

As stated earlier the first step in the compilation is typically the preprocessor.  The preprocessor is a text processing program with vary little relation to C/C++ syntax.

The preprocessor does things like include header files, macro expansion, and conditional compilation.

Could we write a program in C/C++ without using the preprocessor at all?  We actually can, but doing might require a lot of code duplication.  Since our initial program was so simple, let's re-write it without using the preprocessor at all.

```c++
int foo(); // not actually needed

int foo()
{
        return 42;
}
```

```c++
int foo(); // this forward declaration is copied

int main()
{
        return foo();
}
```

Now let's use GCC to produce expanded source files.  These files should closely resemble what we just did by hand.

We can use the -E -P gcc flags to examine the preprocessor output. For example:

```
[]$ g++ -E -P foo.cpp
```
```c++
int foo();

int main()
{
 return foo();
}
```

If we attempt to get an expanded source file for something even as simple as "hello world" we end up with a surprisingly large file.


```c++
// hello.cpp

#include <iostream>

void main() {
	std::cout << "hello world" << std::endl;
}
```

```
[]$ g++ -E hello.cpp > hello-pre.cpp
```

[output](hello-pre.html)

This produces about 356 pages of source code!!!  

Why is it so big?  Well keep in mind that all include files are inserted recursively.  In our example, main.cpp only included foo.hpp.  But what if foo.hpp also included something and then that included something else?

Lucky for us even though hello.cpp expands to almost half a megabyte on disk, our computers and compilers are fast enough that we don't need to wait too long for this program to compile.  As programs grow larger these times balloon quickly.  Large projects can end up taking hours to compile.  Much of this time is due large expanded source files.  The worst part is that frequently most of the code in the expanded source file isn't even needed.  When we look at headers later we will examine techniques that avoid these problems.

The expanded source file is sometimes referred to as a *translation unit*.  From the C++ standard: A translation unit consists of the contents of a single source file, plus the contents of any header files directly or indirectly included by it, minus those lines that were ignored using conditional preprocessing statements.  A translation unit is so named because it is the basic unit that can be translated from the source language (c/c++) to some target language.

### Errors

What kinds of errors can we get during the preprocessor phase?

The most common error is probably attempting to include a header file that doesn't exist:

```c++
// not-found.cpp

#include "amelia-earhart.hpp" // file doesn't exist
```
```
[]$ g++ -E not-found.cpp
not-found.cpp:1:52: error: amelia-earhart.hpp: No such file or directory
```

Another problem is attempting to include oneself:

```c++
// resursive.cpp

#include "recursive.cpp" // include oneself
```

```
[]$ g++ -E recursive.cpp 
recursive.cpp:1:25: error: #include nested too deeply
```

## Compiler

The next phase is the compilation phase.  This phase takes each expanded source file from the previous stage and creates an assembly language file for the target platform.

We can use the -S gcc flag to examine the assembly language output.  For example:

```
[]$ g++ -S foo.cpp
```

By default this command will make a file with the same name as the input, but with the .s extension.  In our case 'foo.s'.

foo.s
```asm
	.file	"foo.cpp"
	.text
.globl _Z3foov
	.type	_Z3foov, @function
_Z3foov:
.LFB0:
	.cfi_startproc
	.cfi_personality 0x3,__gxx_personality_v0
	pushq	%rbp
	.cfi_def_cfa_offset 16
	.cfi_offset 6, -16
	movq	%rsp, %rbp
	.cfi_def_cfa_register 6
	movl	$42, %eax
	leave
	.cfi_def_cfa 7, 8
	ret
	.cfi_endproc
.LFE0:
	.size	_Z3foov, .-_Z3foov
	.ident	"GCC: (GNU) 4.4.7 20120313 (Red Hat 4.4.7-4)"
	.section	.note.GNU-stack,"",@progbits
```

If you have any experience with Intel assembly this file shouldn't look too strange.  Looking directly at the assembly listing can be useful for understanding the code generated by the compiler, but most of the time the assembly listing is only an academic excersise.  We're usually more interested in the output from the next stage of the compilation process (assembly).

### Errors

During the compilation phase any language error (syntax, semantic) in the expanded source file will be reported as an error.  The compiler will verify that the expanded source file is legal under the rules of the programming language.

```c++
// language-error.cpp

int something() {
  return; // should return an int
}
```
```
[]$ g++ -S language-error.cpp
language-error.cpp: In function ‘int something()’:
language-error.cpp:2: error: return-statement with no value, in function returning ‘int’
```

## Assembler

In the next phase the assembly language from the previous stage is assembled into relocatable machine code.  While the assembly language from the previous step is interesting from a more academic perpective, the output from this phase is extremely practical.

The gcc -c flag stops the overall compilation process after the assembler.

```
[]$ g++ -c foo.cpp
```

If foo.cpp can be preprocessed, compiled, and assembled correctly then by default an output file with the same name as the input but having the .o extension (foo.o in this case) and containing the relocatable object code will be created.

We can also create main.o file from main.cpp:

```
[]$ g++ -c main.cpp
```

Notice also that each c/c++ translation unit is built independently from every other file.  This generally allows builds to be paralleled effectively--each translation unit can be built into an object file independently.

Given the .o files we can link the executable later.  This means that if we change a single .cpp file, we can rebuild that single file and then re-link the executable without rebuilding every single translation unit.  This type of build is frequently referred to as an *incremental build* and can save a tremendous amount of time when working on large projects.

### Errors

Since the assembly language is generated by the compiler, we generally don't expect errors in this phase, but it could be possible if there were bugs in the compiler.

## Linker

Linking is the last step of the build process.  During the link process all the object files from the previous phase are combined into a single executable.

Let's use gcc to link together our program's object files into a single executable:

```
[]$ g++ foo.o main.o
```

This command will create link the two object files into a single executable (its default name is a.out).

We can run this program and then use $? and echo to display the program's status code (return value from main).

```
[]$ .a.out
[]$ echo $?  # print exit code
42
```

Now let's assume that we later discover that there was a bug in our foo function.  Instead of returning 42 it was supposed to return 43.  If we edit the implementation of foo within foo.cpp to return 43, we can now rebuild our executable using the following commands:

```
[]$ g++ -c foo.cpp    # re-build foo.o
[]$ g++ foo.o main.o  # re-link executable (a.out)
[]$ ./a.out           # run executable
[]$ echo $?           # print exit code
43
```

Notice that since nothing in the main.cpp translation unit we didn't need to rebuild main.o.  We simply re-built the part that did change (foo.cpp) and re-linked that part to create a new executable.

### Errors

Linker errors are frequently the most frustrating type of bug.  Linker errors are caused by problems when combining the .o files into a single executable.

#### Undefined Reference

A common linking problem is an undefined reference.

```c++
// undefined-reference.cpp

int missing(); // declared, but where is this defined?

int main()
{
	return missing();
}
```

Let's compile undefined-reference.cpp:

```
[]$ g++ -c undefined-reference.cpp
[]$
```

This translation unit compiled to undefined-reference.cpp.o just fine.

Now let's try to link the undefined-reference.o:

```
[]$ g++ undefined-reference.o
undefined-reference.o: In function `main':
missing-defintion.cpp:(.text+0x5): undefined reference to `missing()'
collect2: ld returned 1 exit status
```

The linker reports an error because it can't find a definition for the function 'missing()' within the object files it is linking.

##### Missing Entry Point

Not defining an entry point for the application is also a linker error.  The linker will look for a function named main (by default) somewhere in the .o files being linked.  If appropriate entry point can be found then the linker will report an error.

```c++
// where-is-main.cpp

void a()
{
}

void b()
{
}
```

```
[]$ g++ -c where-is-main.cpp    # works fine
[]$ g++ where-is-main.o
/usr/lib/gcc/x86_64-redhat-linux/4.4.7/../../../../lib64/crt1.o: In function `_start':
(.text+0x20): undefined reference to `main'
collect2: ld returned 1 exit status
```

A missing entry point is basically just a undefined reference, but it is slightly different as the function is never actually referenced by your code.  It is referenced by the linker itself.

#### Multiply Defined Symbols 

The compiler prevents us from defining the same function more than once within a single translation unit, but what if we define the same function in two different translation units?

```c++
// dup-1.cpp
int dup() { return 42; }
```

```c++
// dup-2.cpp

int dup() { return 42; }

int main() { return dup(); }
```

Now let's try compiling each of these 

```
[]$ g++ -c dup-1.cpp
[]$ g++ -c dup-2.cpp
[]$ g++ dup-1.o dup-2.o
dup-2.o: In function `dup()':
dup-2.cpp:(.text+0x0): multiple definition of `dup()'
dup-1.o:dup-1.cpp:(.text+0x0): first defined here
collect2: ld returned 1 exit status
```

## Execution

Execution isn't actually part of the compilation process, but presumably the point of building an executable is that we wanted to run it.  Assuming all the previous build steps worked correctly, we should end up with an executable file.  With gcc it gives this executable the name 'a.out' by default.  We can run the program at the Linux command prompt as follows:

```
[]$ ./a.out
```

### Errors

Just because we got our program to build doesn't mean that it will execute without error.  Several kinds of errors might happen at run-time.

#### Fatal Error

The program crashes.  The following are some examples of fatal errors:

##### Divide by Zero

```c++
#include <iostream>
using namespace std;

int main() {
        cout << "enter a: ";

        int a;
        cin >> a;

        cout << "enter b: ";

        int b;
        cin >> b;

		// should make sure that b != 0

        cout << "a / b is: " << a / b << endl;

        return 0;
}
```

```
# output 

[]$ ./a.out
enter a: 4
enter b: 2
a / b is: 2
[]$ ./a.out
enter a: 4
enter b: 0
Floating point exception
```

##### Segmentation Fault

If a program attempts to access memory that it does not own this results in a segmentation fault.  On Windows this type of error is refered to as an access violation.

```c++
// seg-fault.cpp

int main() {
        int *a = 0;

        int b = *a; // dereferences 0

        return 0;
}
```

output:

```
[]$ g++ seg-fault.cpp
[]$ ./a.out
Segmentation fault
```

If a C/C++ program goes into an infinite recursion this typically results in a run-time error.  On Windows this type of error is called a stack overflow.

```c++
// inf-recursion.cpp

int main() {
        return main();
}
```

output:

```
[l]$ g++ inf-recursion.cpp
[]$ ./a.out
Segmentation fault
```

#### Logical Error

The program doesn't do what the programmer intended.

The following program is intended to display the lyrics to a popular song.

```c++
// 99-bottles.cpp

#include <iostream>
using namespace std;

int main() {
        int bottles = 99;

        while (bottles > 0) {
                cout << bottles << " bottles of root beer on the wall," << endl
                     << bottles << " bottles of root beer!" << endl
                     << "take one down, pass it around," << endl
                     << --bottles << " bottles of root beer on the wall!" << endl;
        }

        return 0;
}

```

output (truncated)

```
98 bottles of root beer on the wall,
98 bottles of root beer!
take one down, pass it around,
98 bottles of root beer on the wall!

...

1 bottles of root beer on the wall,
1 bottles of root beer!
take one down, pass it around,
1 bottles of root beer on the wall!

0 bottles of root beer on the wall,
0 bottles of root beer!
take one down, pass it around,
0 bottles of root beer on the wall!
```

What's the problem?  It has to do with the behavior of the pre and post fix decrement (and increment) operators.  First let's look at a corrected verison:

```c++
#include <iostream>
using namespace std;

int main() {
        int bottles = 99;

        while (bottles > 0) {
                cout << bottles << " bottles of root beer on the wall," << endl
                     << bottles << " bottles of root beer!" << endl
                     << "take one down, pass it around," << endl;
                cout << --bottles << " bottles of root beer on the wall!" << endl;
        }

        return 0;
}
```

output (truncated):

```
99 bottles of root beer on the wall,
99 bottles of root beer!
take one down, pass it around,
98 bottles of root beer on the wall!

...

2 bottles of root beer on the wall,
2 bottles of root beer!
take one down, pass it around,
1 bottles of root beer on the wall!

1 bottles of root beer on the wall,
1 bottles of root beer!
take one down, pass it around,
0 bottles of root beer on the wall!
```

So what was the original problem?  The explaination involves several aspects of C++ that we will cover later in the course, so I'll skip the detailed explanation for now.  The basic idea is that the calls to cout << are basically function calls.  Given that consider the following:

```c++
#include <iostream>
using namespace std;

int sub(int const a, int const b) {
	return a - b;
}

int main() {
	int a = 1;
	cout << sub(a--, a) << endl;
}
```

The output of this program depends on the order of the evaluation of the parameters.  If the parameters are evaled left to right, then we would get 1.  If they are evaluated right to left we would get 0.  Most compilers evaluate left to right, but technically this is undefined by the language so it could work either way.

So when all entire cout was written as a single chain the side effect of --bottles was completed before the other values were passed to cout.

Since this behavior is technically undefined a better solution would be to re-write it as follows:

```c++
#include <iostream>
using namespace std;

int main() {
	int const totalBottles = 99;

	for (int i = 0; i < totalBottles; ++i) {
		int const bottles = totalBottles - i;

		cout << bottles << " bottles of root beer on the wall," << endl
		     << bottles << " bottles of root beer!" << endl
		     << "take one down, pass it around," << endl
		     << bottles - 1 << " bottles of root beer on the wall!" << endl << endl;
	}

	return 0;
}
```

A few things to remember:
- logical errors are usually the most common and difficult problems to find
   - you might not see them at all (the faulty code isn't executed)
   - you might not understand what is wrong or where the problem is


----------

[feedback](https://www.surveymonkey.com/s/WKBZ28H)