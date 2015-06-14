---
title: Introduction to C++ 11
---
# Introduction to C++ 11

The C++ language and standard library continue to evolve and progress.  The C++ version names take the year that the standard was approved, in this case 2011.

The language is standardized by a committee of about 250 people.  These people attend meetings, participate in discussions, and submit proposal papers.  The final standards are voted on by around 20 standards bodies from around the world.

C++11 was a very large change to the C++ language and libraries.  There were a huge number of features added.  Programming in C++11 really feels like practically a different programming language.  We won't be able to cover all the new features in a single class (or even in a single semester actually), but this is meant to give you a taste.  It is also meant to give you a chance to try out a few of the features to make sure that your compiler is working with C++11.

Since C++11 has been out for a few years now, compiler support is getting to be pretty good with the latest compiler versions.  However it is far from complete or universal.  Many features still do not work and are not implemented on some compilers.  I'll attempt to focus on those features I've personally found most useful.  I'll try to avoid features that aren't supported on the compilers that I've used.

*Note on terminology:*

While the C++11 standard was being defined the authors did not know how long it would take to finish, therefore they did not have a name (remember the name is based on the year the standard is approved).  Instead they simply used X as a placeholder for the year it would be released.  The committee believed the work would be finished before 2009, so they called it C++0x while it was under development.  Unfortunately they did not finish their work before 2010.  Bjarne Stroustrup later claimed that x was a hexadecimal number (i.e. C++0B, aka C++11).  This is why you'll sometimes see C++11 referred to as C++0x.

## Features

Let's look at a couple new language features in C++11.  This will be a very basic and extremely incomplete introduction.  Basically this will give you a couple things you can try to make sure that your compiler supports C++11 syntax.

### nullptr

C/C++ traditionally used the macro NULL to represent a pointer that pointed to nothing.  NULL is defined a the integer constant 0.  This causes the potentially confusing issue to arise:

```c++
void func(int a);
void func(int *a);

func(NULL); // calls func(int)
```

nullptr is a replacement for NULL.  It is always a pointer.  Anywhere you used to use NULL, you can now use nullptr and avoid some of the pitfalls of NULL.

### auto

C++11 supports a feature called type inference.  This allows us to avoid having to explicitly state the type of a variable when it can be inferred by the initializer.  For example:

```c++
auto a = 42;    // a's type is int
auto b = 4.2;   // b's type is double
auto c = false; // c's type is bool
auto d = foo(); // d's type is whatever foo's return type is
auto e = "kp";  // e's type is char const *
```

This feature is especially useful when dealing with generic code (templates) and extremely long type names (which often occur in the C++ standard libraries).

### Range-Based for Loop

If we wanted to enumerate the elements of an array we would normally write it as follows:

```c++
int const array[] = { 1, 2, 3 };

for (int i = 0; i < 3; ++i) {
	int const &element = array[i];
	cout << element << " ";
}
```

C++11 provides a less verbose, more declarative syntax for this kind of loop:

```c++
int const array[] = { 1, 2, 3 };

for (auto const &element : array) {
	cout << element << " ";
}
```

The need to specify the index variable is eliminated.  Keep in mind however that if you need access to the index variable you can't use the range-based for loop syntax.

This feature can actually used to enumerate any kind of collection.  If we had written a custom linked list class for example, we could add the ability for the range-based for loop to work with our custom type.  The collections in the standard library support this new for loop as well.

## Using C++11 on CSC Linux Server

### C++11 Toggle

GCC hides C++11 support behind a compiler flag.  This means that by default the compiler will not support C++11 features.  If we want to enable C++11 we need to pass a command line flag telling the compiler that we want it to allow C++11 features.

The flag used to be called **-std=c++0x** (GCC 4.3-4.6), but now is **-std=c++11** (4.7 and later).


### GCC Versions

The default version of gcc on the build server is 4.4.7.  Although this does have some level of C++11 support, more features are available with newer versions of the compiler.  The server also has GCC 4.7.2, but it is not the default.  For example:

```bash
[]$ g++ --version
g++ (GCC) 4.4.7 20120313 (Red Hat 4.4.7-4)
```

```bash
[]$ g++-472 --version
g++-472 (GCC) 4.7.2 20121015 (Red Hat 4.7.2-5)
```

So if we want to build our programs with a newer GCC on the Linux server, we can use g++472.

For example:

```bash
[]$ g++-472 -std=c++11 test.cpp
```


### Working with CMake

There are a few additional details we need to cover to get your programs to work with both C++11 and CMake.  The main issue is that we need to tell CMake to use the newer version of GCC.  CMake will look for the normally will look for the default version of GCC, but we can tell it a specific version.

If we run the following we can see where g++-472 is installed on the Linux server:

```bash
[]$ whereis gcc-472
gcc-472: /usr/local/bin/gcc-472
```

When we call CMake for the first time we can tell it to initialize some variables by using a -D flag.  We use this the very first time (along with the -G flag to specify the generator).

To make this easy, I usually do this as a shell script:

```bash
# generate-projects-make.sh

mkdir -p project
cd project
cmake -G "Unix Makefiles" -D CMAKE_C_COMPILER=/usr/local/bin/gcc-472 -D CMAKE_CXX_COMPILER=/usr/local/bin/g++-472 ../
cd ../
```

The above script makes a project directory, changes to it, runs cmake with the make generator, tells cmake where GCC 4.7.2 is located, and then returns to the parent directory where we started.  This script is specific for building on the CSC Linux server, but the rest of this tutorial will apply for setting up a project for C++11 regardless of your platform.

If we want to run this script, we need to change permissions on the file to allow execute permissions.

```bash
[]$ chmod +x generate-projects-make.sh
```

Now we can run this script to setup our projects.  Normally you would only need to run this once.  This is nice if you want to give this project to somebody else (let's say your instructor), or if you want to rebuild your project (just delete the project directory) and re-run the script.

But before we run our script we need to actually have a CMakeLists.txt to specify what we want to build.  For this example we'll just have a really basic program with an executable that has a single source file:

```cmake
# CMakeLists.txt

cmake_minimum_required(VERSION 2.8)

project(hello-cpp-11)

include(EnableCpp11.cmake) # copy this file's contents

add_executable(hello-cpp-11
  main.cpp
)
```

The only thing new here is the include() statement.  This is basically CMake's version of what we do with the C preprocessor **#include "filename"**.  We're using that to include another cmake file.  This is a file I wrote that also needs to be in our project.  Let's have a look:

```cmake
# EnableCpp11.cmake

include(CheckCXXCompilerFlag) # standard cmake module
include(CheckCCompilerFlag)   # standard cmake module

# enable C++11 support
check_cxx_compiler_flag("-std=c++11" has_cxx_11)
if (has_cxx_11)
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
endif()
set(CMAKE_XCODE_ATTRIBUTE_CLANG_CXX_LANGUAGE_STANDARD "c++11")
set(CMAKE_XCODE_ATTRIBUTE_CLANG_CXX_LIBRARY "libc++")

check_cxx_compiler_flag("-std=c++0x" has_cxx_0x)
if (has_cxx_0x)
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
endif()
```

This CMake code will use the CheckCxxCompilerFlag library (a built-in part of CMake) to figure out the appropriate C++11 toggle for whatever compiler we're using.  Remember we need to tell GCC that we want to enable C++11 support.  Not that this code will also set clang C++11 toggles needed for Xcode.  If your compiler needs different toggles, they could be put here.

Finally let's put some C++11 code in our main.cpp file:

```c++
#include <iostream>
using namespace std;

int main() {
        int const array[] = { 1, 2, 3 };
        for (auto const &element : array) {
                cout << element << " " << endl;
        }

        return 0;
}
```

To run our script:

```bash
[]$ ./generate-projects-make.sh
-- The C compiler identification is GNU 4.7.2
-- The CXX compiler identification is GNU 4.7.2
-- Check for working C compiler: /usr/local/bin/gcc-472
-- Check for working C compiler: /usr/local/bin/gcc-472 -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working CXX compiler: /usr/local/bin/g++-472
-- Check for working CXX compiler: /usr/local/bin/g++-472 -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Performing Test has_cxx_11
-- Performing Test has_cxx_11 - Success
-- Performing Test has_cxx_0x
-- Performing Test has_cxx_0x - Success
-- Configuring done
-- Generating done
-- Build files have been written to: project
```

Note that CMake found GCC 4.7.2 as our compiler.  Also note that the appropriate c++11 compiler toggle was determined.  Now that we have our have our basic project setup complete we can built our program using CMake:

```bash
[]$ cmake --build project/
Scanning dependencies of target hello-cpp-11
[100%] Building CXX object CMakeFiles/hello-cpp-11.dir/main.cpp.o
Linking CXX executable hello-cpp-11
[100%] Built target hello-cpp-11
```

To run our program:

```c++
[]$ project/hello-cpp-11
1
2
3
```

If we want to delete our generated project files (where the make files, object files, cmake files are located) we can just delete the project directory:

```bash
[]$ rm -r project
```


## Using C++11 on Visual Studio

Visual Studio supports C++11 by default.  There is no need to add any special flags or toggles.

Visual Studio support for C++11 is somewhat limited compared to GCC.  The newest version (2013) has the best level of support.  I'll generally avoid discussing features that don't work across the major compilers.

For C++11 I would highly recommend using at least Visual Studio 2013.

If we want to take our previous code from the CSC Linux Server we just need to write a new script to generate projects.  Now we're on Windows so we'll make a .bat file:

```bat
REM generate-projects-vs13.bat

mkdir project
cd project
cmake -G "Visual Studio 12" ../
cd ../
```

Now we can run this bat file on Windows and create our Visual Studio 2013 solution and projects.

```cmd
generate-projects-vs13.bat
REM generate-projects-vs13.bat
mkdir project
cd project
cmake -G "Visual Studio 12" ../

-- The C compiler identification is MSVC 18.0.31101.0
-- The CXX compiler identification is MSVC 18.0.31101.0
-- Check for working C compiler using: Visual Studio 12
-- Check for working C compiler using: Visual Studio 12 -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working CXX compiler using: Visual Studio 12
-- Check for working CXX compiler using: Visual Studio 12 -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Performing Test has_cxx_11
-- Performing Test has_cxx_11 - Failed
-- Performing Test has_cxx_0x
-- Performing Test has_cxx_0x - Failed
-- Configuring done
-- Generating done
-- Build files have been written to: 

cd ../
``` 

## Using C++11 on Xcode

For Xcode we just need another script to generate Xcode projects:

```bash
# generate-projects-xcode.sh
mkdir -p project
cd project
cmake -G "Xcode" ../
cd ../
```

Note that we'll have to make sure that this script is executable as on Linux (described above).

Output:

```
$ ./generate-projects-xcode.sh 
-- The C compiler identification is Clang 6.0.0
-- The CXX compiler identification is Clang 6.0.0
-- Check for working C compiler using: Xcode
-- Check for working C compiler using: Xcode -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working CXX compiler using: Xcode
-- Check for working CXX compiler using: Xcode -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Performing Test has_cxx_11
-- Performing Test has_cxx_11 - Success
-- Performing Test has_cxx_0x
-- Performing Test has_cxx_0x - Success
-- Configuring done
-- Generating done
-- Build files have been written to: project
$ 
```