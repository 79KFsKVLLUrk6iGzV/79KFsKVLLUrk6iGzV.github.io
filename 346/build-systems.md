---
title: Build Systems
---
# Build Systems

A build system is a way of automatically running the steps of the build process.

## A Manual Build Process

Let's make a simple program and manually do the steps required to create our executable.  Since full (non-incremental) builds will not work for most real-world programs, we'll use incremental builds (even though our with our small example incremental builds would be fast enough).

```c++
// inc.hpp
#pragma once
int inc(int const a);
```

```c++
// inc.cpp
#include "inc.hpp"

int inc(int const a) {
	return a;
}
```

```c++
// add.hpp
#pragma once
int add(int const a, int const b);
```

```c++
// add.cpp
#include "inc.hpp"

int add(int const a, int const b) {
	int sum = a;
	for (int i = 0; i < b; ++i)
		sum = inc(sum);
	return sum;
}
```

```c++
// main.cpp
#include "inc.hpp"
#include "add.hpp"

int main() {
	return add(inc(1), inc(1));
}
```

Since we've never built this program before let's compile each of the source files to .o files:

```console
[]$ g++ -c inc.cpp add.cpp main.cpp
```

Since everything worked we have 3 .o files:

```console
[]$ ls
add.cpp  add.hpp  add.o  inc.cpp  inc.hpp  inc.o  main.cpp  main.o
```

Next we'll link our .o files into an executable:

```console
[]$ g++ add.o inc.o main.o
[]$ ls
add.cpp  add.hpp  add.o  a.out  inc.cpp  inc.hpp  inc.o  main.cpp  main.o
```

Now we have our executable a.out.

```console
[]$ ./a.out
[]$ echo $?
1     # oops!! should be 4, what's wrong?
```

Found it!  There's a mistake in inc.cpp--inc is supposed to return a + 1, not a.  Let's correct this mistake and rebuild:

```c++
// inc.cpp (altered)
#include "inc.hpp"

int inc(int const a) {
	return a + 1;
}
```

Now let's recreate the .o file for inc.cpp:

```console
[]$ g++ -c inc.cpp           # preprocess, compile, assemble
[]$ g++ add.o inc.o main.o   # link
```

Now we have our updated executable:

```console
[]$ ./a.out
[]$ echo $?
4
```

Looks better.  Now the boss shows up and he thinks he found out that Sue over on the other programming team needs an inc function also, but she needs an increment by some arbitrary amount, not just one.  Since we're sharing code wherever possible he wants us to use the same inc function.  Let's update our program again to meet our boss's demands:

```c++
// inc.hpp (altered)
#pragma once
int inc(int const a, int const incrementBy = 1);
```

```c++
// inc.cpp (altered again)
#include "inc.hpp"

int inc(int const a, int const incrementBy) {
	return a + incrementBy;
}
```

Now let's rebuild our executable again.

```console
[]$ g++ -c inc.cpp           # preprocess, compile, assemble
[]$ g++ add.o inc.o main.o   # link
add.o: In function `add(int, int)':
add.cpp:(.text+0x23): undefined reference to `inc(int)'
main.o: In function `main':
main.cpp:(.text+0xf): undefined reference to `inc(int)'
main.cpp:(.text+0x1b): undefined reference to `inc(int)'
collect2: ld returned 1 exit status
```

Well that didn't work so well!  What is wrong?

Remember that header files get expanded in the source files in which they are included.  We recreated the .o file for inc.cpp, but since we also updated inc.hpp we need to remember to update all the .cpp files that include inc.hpp.  In our case that is add.cpp and main.cpp.

> In our small example program it is pretty easy to figure out which files include inc.hpp, but for a real world program maintained by multiple developers this would quickly become difficult.

```console
[]$ g++ -c add.cpp main.cpp
[]$ g++ add.o inc.o main.o
[]$ ./a.out
[]$ echo $?
4
```

## Why Bother?

At this point you're probably thinking that we're doing a lot of work to achieve this incremental build, and you'd be right.  We could just do the following:

```
[]$ g++ add.cpp inc.cpp main.cpp
```

Run that and everything seems to work easy.  That might work for now, but it won't work when your programs get larger.  What happens when your program gets so large that you need to wait 10 minutes for every full build.  Do you really want to wait 10 minutes after every minor change?  Do you think your boss will want you sitting around for 10 minutes every time you need to build?

Maybe you don't believe me or don't care because it doesn't affect you yet, but build speed isn't the only problem:

- How do we know what source files should be built?  Even in a school project you could end up with 10-20 files.  Do you really want to type all of those files in every time you need to compile?

- What if you want to use tools like IDEs (e.g. Visual Studio, Xcode) for development?

- What if you want to see progress of your build?

- What if you work with other programmers on a team?  How do they know what to build?  What if different people on a team use different IDEs?

- What if you want to build your program on different operating systems?

- What if you want to use 3rd party libraries?  How do you document all the flags required to have your program build and/or link to the 3rd party code.

- What if you need a code generator to create some of the program automatically?

- What if your teacher is a jerk and wants you to use them?

A build system can help address these problems.

Builds are boring, repetitive, time-consuming, and easy to screw up.  Why do this work ourselves when we can make our computers do the work for us?

## CMake

Hopefully by now you're totally on board that build systems are a great idea and you're ready to learn all about them.  If not then listen anyway because I'm going to make you use CMake for your assignments.

**Disclaimer:**  This isn't a class about CMake, and I'm not going to cover it in great detail.  I will cover enough for you to be able to use it for your assignments, but I'm not even going to scratch the surface of what you can do with CMake.

### Basics

**What is CMake?**

- CMake is a build system (or more properly a meta build system)
- It is cross-platform (that's what the c means); i.e. it runs on many platforms and it can be used to build cross-platform software
- It is free/open-source
- It is compatible with various IDEs (Eclipse, Xcode, Visual Studio), or it can be used command line
- It has a fairly easy to use syntax

**How can I get CMake?**

Download an installer from the [CMake website](http://www.cmake.org/download/).

It can also be built from sources.

It is already installed on the Linux server.

**I've heard of make.  Is CMake make?**

No.  CMake and make both are build systems and can serve a similar role, but they are different tools.  make is older and still used for many projects, but CMake is generally considered more intuitive than make and becoming quite popular.  CMake also natively supports a wide variety of IDEs and is more easily used on non-Unix systems.  CMake is especially useful for cross-platform development with C and C++.

**What do you mean meta-build system?**

CMake is actually a tool that generates build systems.  It has generators for a wide variety of build systems.  On Linux we might use CMake to generate make files to build our programs.  On Windows, CMake can generate a Visual Studio solution, and then Visual Studio builds our program.

### Overview

To use CMake you write one or more project files.  These files (named CMakeLists.txt) contain a description of what you want to build.  These CMakeLists.txt are scripts written in the CMake scripting language.  We then invoke the CMake executable passing the directory of our root CMakeLists.txt file and the desired generator.

What we do next depends on the generator.  If we used the make generator, then we run make with the generated Makefile.  If we used the Visual Studio generator we open the generated solution file with Visual Studio.

If we use an IDE like Visual Studio CMake will even add a little smarts to the generated solution to check for changes to our CMakeLists.txt files when we build.  If there are any changes the solution/project files will be regenerated and Visual Studio will prompt us to reload the project.

Note however that this generation is one way.  If we make a project change through Visual Studio (e.g. add a file to the project) using the IDE instead of editing the CMakeLists.txt, then Visual Studio will save these changed to our *generated* project files.  Next time we regenerate our project any changes will be lost.

Besides that one caveat, using CMake with an IDE is pretty much just like using it normally.  We can use all the editing and debugging tools as we normally would.  We can easily see the source/header files within the IDE.

### Demo

Let's recreate what we did earlier but this time using CMake.  First we'll start with the same program:

```c++
// inc.hpp
#pragma once
int inc(int const a);
```

```c++
// inc.cpp
#include "inc.hpp"

int inc(int const a) {
	return a;
}
```

```c++
// add.hpp
#pragma once
int add(int const a, int const b);
```

```c++
// add.cpp
#include "inc.hpp"

int add(int const a, int const b) {
	int sum = a;
	for (int i = 0; i < b; ++i)
		sum = inc(sum);
	return sum;
}
```

```c++
// main.cpp
#include "inc.hpp"
#include "add.hpp"

int main() {
	return add(inc(1), inc(1));
}
```

Now we need to write a CMakeLists.txt file.  Here we'll introduce some basic CMake commands.  We only need 3 to get started.

```cmake
# tell cmake the min version, this prevents a warning
cmake_minimum_required(VERSION 2.8)

# give a name for the project
project(math)

# tell cmake that we want to make an executable
# the first argument is the name of the excutable
# the rest of the args are the files in our project
# note: the header files are optional, but they are a good idea for IDEs 
add_executable(hello
  inc.hpp
  inc.cpp
  add.hpp
  add.cpp
  main.cpp
)
```

Now we run cmake.  By default cmake will look for a CMakeLists.txt file in the current working directory.  We also need to tell cmake which generator to use.

If we want to see the generators supported we can run cmake --help.  The generators listed will depend on the OS and what programming tools are installed on the machine.

```console
[]$ cmake --help   ## (Truncated) ##
cmake version 2.8.12.2
Usage

  cmake [options] <path-to-source>
  cmake [options] <path-to-existing-build>

Options
  -G <generator-name>         = Specify a build system generator.
                                generator.
  --build <dir>               = Build a CMake-generated project binary tree.

Generators

The following generators are available on this platform:
  Unix Makefiles              = Generates standard UNIX makefiles.
  Ninja                       = Generates build.ninja files (experimental).
  CodeBlocks - Ninja          = Generates CodeBlocks project files.
  CodeBlocks - Unix Makefiles = Generates CodeBlocks project files.
  Eclipse CDT4 - Ninja        = Generates Eclipse CDT 4.0 project files.
  Eclipse CDT4 - Unix Makefiles
                              = Generates Eclipse CDT 4.0 project files.
  KDevelop3                   = Generates KDevelop 3 project files.
  KDevelop3 - Unix Makefiles  = Generates KDevelop 3 project files.
  Sublime Text 2 - Ninja      = Generates Sublime Text 2 project files.
  Sublime Text 2 - Unix Makefiles
                              = Generates Sublime Text 2 project files.
```

Let's use the Unix Makefiles generator.

```console
[]$ ls
add.cpp  add.hpp  CMakeLists.txt  inc.cpp  inc.hpp  main.cpp
[]$ cmake -G "Unix Makefiles"
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
-- Build files have been written to: 
[]$ ls
add.cpp  CMakeCache.txt  cmake_install.cmake  inc.cpp  main.cpp
add.hpp  CMakeFiles      CMakeLists.txt       inc.hpp  Makefile
```

Notice that cmake generated CMakeCache.txt (a file used to remember variables that allow faster generation), cmake_install.cmake, and critically Makefile.  Makefile is a Unix make file that we can pass to the make utility to build our program for us.

One of the things cmake caches for us is the generator used.  This means that we only need to specify the generator the first time we run cmake for a project (or after deleting CMakeCache.txt).

CMake also does things like locate the compiler.  Again this stuff takes a little time, so it stores all that information into the cache so that it doesn't have to do that work again next time.  This helps make builds faster when using cmake.

We could run run the make command now.  make will find Makefile in the current directory and build the project for us, but we don't need to run make at all, we can just tell cmake to run make for us.  Running the build this way is better than manually running make because if we make any changes to our CMakeLists.txt files cmake will automatically regenerate the Makefile before calling make for us.

Let's build our project using cmake:

```console
[]$ cmake --build .
Scanning dependencies of target hello
[ 33%] Building CXX object CMakeFiles/hello.dir/inc.cpp.o
[ 66%] Building CXX object CMakeFiles/hello.dir/add.cpp.o
[100%] Building CXX object CMakeFiles/hello.dir/main.cpp.o
Linking CXX executable hello
[100%] Built target hello
```

Notice the build output.  We can see the progress of our build.  Also notice how each cpp file is compiled.

Looking again at the files in our directory:

```console
[]$ ls
add.cpp  CMakeCache.txt  cmake_install.cmake  hello    inc.hpp   Makefile
add.hpp  CMakeFiles      CMakeLists.txt       inc.cpp  main.cpp
```

Notice that the hello executable now sits in our directory.

CMake also added a CMakeFiles directory.  Looking at the files generated:

```console
[]$ ls CMakeFiles/
2.8.12.2                         CMakeTmp        progress.marks
cmake.check_cache                hello.dir       TargetDirectories.txt
CMakeDirectoryInformation.cmake  Makefile2
CMakeOutput.log                  Makefile.cmake
[]$ ls CMakeFiles/hello.dir/
add.cpp.o          CXX.includecache  depend.make  link.txt
build.make         DependInfo.cmake  flags.make   main.cpp.o
cmake_clean.cmake  depend.internal   inc.cpp.o    progress.make
```

The key thing to notice is the .o files for each of our source files.

If we run our executable, we notice that it has the same bug as the original (inc doesn't work properly, so the program returns 1 instead of 4):

```console
[]$ ./hello
[]$ echo $?
1
```

Let's fix the problem again:

```c++
// inc.cpp (altered)
#include "inc.hpp"

int inc(int const a) {
	return a + 1;
}
```

Now let's do an incremental build again.  Look how easy it is this time:

```compile
[$ cmake --build .
Scanning dependencies of target hello
[ 33%] Building CXX object CMakeFiles/hello.dir/inc.cpp.o
Linking CXX executable hello
[100%] Built target hello
```

Notice what just happened.  CMake first scans the files for changes.  It noticed that inc.cpp changed, so it rebuilds inc.cpp.o.  Since that was the only change it relinks the other .o files from the previous build.  This is exactly the same process we did manually before, but now it is completely automated by our build system.

Let's make another change.  This time let's do the change to inc that caused both the header and source files to change.  Remember that since we're updating a header, every source file that includes that header will need to be rebuilt.

```c++
// inc.hpp (altered)
#pragma once
int inc(int const a, int const incrementBy = 1);
```

```c++
// inc.cpp (altered again)
#include "inc.hpp"

int inc(int const a, int const incrementBy) {
	return a + incrementBy;
}
```

Now let's do an incremental build again.  Again, easy:

```console
[]$ cmake --build .
Scanning dependencies of target hello
[ 33%] Building CXX object CMakeFiles/hello.dir/inc.cpp.o
[ 66%] Building CXX object CMakeFiles/hello.dir/add.cpp.o
[100%] Building CXX object CMakeFiles/hello.dir/main.cpp.o
Linking CXX executable hello
[100%] Built target hello
```

Notice this time it scans dependencies and figures out automatically which source files need to be re-built.  In our case it was all three of our source files.  They're built and re-linked into an updated executable.

Again this is way easier.  We don't need to figure out which files all include inc.hpp and make sure we re-build all of them.  We don't need to worry about linker errors because we forgot one of the header files.  We don't need to type all these commands in manually.  Even if the build process takes a while, at least it doesn't require our active attention we can just kick it off and wait for it to complete.

> Again I can't stress enough how important build times are for real-world systems.  Here's some AnandTech benchmarks showing compile times for the Firefox browser on a Intel i7 4960X (this is a $1000 retail, 6-core CPU):

> ![](http://images.anandtech.com/graphs/graph7255/57915.png)

> As you can see compile times can be a real problem.  Faster hardware can help, but even with the fastest available hardware build times can be extremely slow.

#### Visual Studio

Now let's take the same project and build it with Visual Studio.

```console
C:\.>cmake
<truncated>

Generators

The following generators are available on this platform:
  Visual Studio 6             = Generates Visual Studio 6 project files.
  Visual Studio 7             = Generates Visual Studio .NET 2002 project
                                files.
  Visual Studio 10            = Generates Visual Studio 10 (2010) project
                                files.
  Visual Studio 11            = Generates Visual Studio 11 (2012) project
                                files.
  Visual Studio 12            = Generates Visual Studio 12 (2013) project
                                files.
  Visual Studio 7 .NET 2003   = Generates Visual Studio .NET 2003 project
                                files.
  Visual Studio 8 2005        = Generates Visual Studio 8 2005 project files.
  Visual Studio 9 2008        = Generates Visual Studio 9 2008 project files.
  Borland Makefiles           = Generates Borland makefiles.
  NMake Makefiles             = Generates NMake makefiles.
  NMake Makefiles JOM         = Generates JOM makefiles.
  Watcom WMake                = Generates Watcom WMake makefiles.
  MSYS Makefiles              = Generates MSYS makefiles.
  MinGW Makefiles             = Generates a make file for use with
                                mingw32-make.
  Unix Makefiles              = Generates standard UNIX makefiles.
  Ninja                       = Generates build.ninja files (experimental).
  CodeBlocks - MinGW Makefiles= Generates CodeBlocks project files.
  CodeBlocks - NMake Makefiles= Generates CodeBlocks project files.
  CodeBlocks - Ninja          = Generates CodeBlocks project files.
  CodeBlocks - Unix Makefiles = Generates CodeBlocks project files.
  Eclipse CDT4 - MinGW Makefiles
                              = Generates Eclipse CDT 4.0 project files.
  Eclipse CDT4 - NMake Makefiles
                              = Generates Eclipse CDT 4.0 project files.
  Eclipse CDT4 - Ninja        = Generates Eclipse CDT 4.0 project files.
  Eclipse CDT4 - Unix Makefiles
                              = Generates Eclipse CDT 4.0 project files.
  Sublime Text 2 - MinGW Makefiles
                              = Generates Sublime Text 2 project files.
  Sublime Text 2 - NMake Makefiles
                              = Generates Sublime Text 2 project files.
  Sublime Text 2 - Ninja      = Generates Sublime Text 2 project files.
  Sublime Text 2 - Unix Makefiles
                              = Generates Sublime Text 2 project files.
```

Just running cmake on Windows we can see the list of supported targets on this system.  Let's use the newest Visual Studio.

```console
C:\.>cmake -G "Visual Studio 12"
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
-- Configuring done
-- Generating done
-- Build files have been written to: C:/Users/Kevin/Dropbox/CS-346/examples/build-systems
```

Now cmake has generated a Visual Studio solution for us (math.sln).  This name comes from the project name in our top-level CMakeLists.txt file.

```console
add.cpp                      add.hpp
ALL_BUILD.vcxproj            ALL_BUILD.vcxproj.filters
CMakeCache.txt               [CMakeFiles]
CMakeLists.txt               cmake_install.cmake
hello.vcxproj                hello.vcxproj.filters
inc.cpp                      inc.hpp
main.cpp                     math.sln
out.txt                      ZERO_CHECK.vcxproj
ZERO_CHECK.vcxproj.filters     
```

Having a look at the project in Visual Studio:

![](build-systems-cmake-output-vs.png)

Notice that our hello project has all the source/header files defined within CMakeLists.txt.

Now let's build our Visual Studio solution:

```
1>------ Build started: Project: ZERO_CHECK, Configuration: Debug Win32 ------
1>  Checking Build System
1>  CMake does not need to re-run because C:/Users/Kevin/Dropbox/CS-346/examples/build-systems/CMakeFiles/generate.stamp is up-to-date.
2>------ Build started: Project: hello, Configuration: Debug Win32 ------
2>  Building Custom Rule C:/Users/Kevin/Dropbox/CS-346/examples/build-systems/CMakeLists.txt
2>  CMake does not need to re-run because C:\Users\Kevin\Dropbox\CS-346\examples\build-systems\CMakeFiles\generate.stamp is up-to-date.
2>  inc.cpp
2>  add.cpp
2>  main.cpp
2>  Generating Code...
2>  hello.vcxproj -> C:\Users\Kevin\Dropbox\CS-346\examples\build-systems\Debug\hello.exe
3>------ Skipped Build: Project: ALL_BUILD, Configuration: Debug Win32 ------
3>Project not selected to build for this solution configuration 
========== Build: 2 succeeded, 0 failed, 0 up-to-date, 1 skipped ==========
```

