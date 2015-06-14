---
title: Namespaces
---
# Namespaces

C++ supports a feature called namespaces.  The primary purpose of namespaces is to support combining libraries written by different groups into a single program.

## Composition Problems

If you write a whole program yourself you can simply avoid giving two symbols the same name, but what if you want to write a program that composes several libraries written by different teams.  The library authors don't know what programs their library might be used in.

When the libraries are combined together into a single program we could easily end up with a naming conflict.  Keep in mind that real-world libraries are huge (1000s of functions is not uncommon) and real-world programs will likely combine dozens or more 3rd party libraries.

Here would be a simple example:

```c++
// libfoo.hpp

int foo();
```

```c++
// libbar.hpp

int foo();
```

```c++
// kevin's super program

#include <libfoo.hpp>
#include <libbar.hpp>

int main()
{
	foo(); // which foo??  // ERROR: duplicate symbol

}
```

If I wrote all this code myself it would be easy to change.  But if this was in a 3rd party library, it might not be so easy to change.

The function might be used internally in many places in the library.  If I want to change the name in the library I'll have to update it in all those places.

If I make changes to a 3rd party library it becomes more difficult to update to a new version.  I'll need to re-apply my changes to the library each time I update.

If I make changes to a 3rd party library then I need to build the library myself.  This means that I need all the source code for the library.  Not all libraries will provide their source code.  Some libraries are difficult to build from source.  We might prefer to use a pre-built version of the library instead of building it ourselves.  All of this is difficulties may complicate modifying a library.

## Namespaces in C

How does C solve this problem?  Remember in C++ we solve this problem with namespaces (we'll see them soon).  C does not have namespaces.  Understanding how this issue is addressed in C gives us a better understanding of the problem that the designers of C++ had in mind when they created namespaces.

Let's have a look at some C libraries.  These are real-world libraries.  I've used them in actual programs.

### Simple DirectMedia Layer

[SDL](http://www.libsdl.org/) is a library that allows you to create windows, receive keyboard/mouse/controller input, and interface to graphics APIs like OpenGL or DirectX.  Critically it allows you to do this in a cross-plaform manner (normally the APIs that allow you to do these things are platform-specific).  It is used by many cross-platform games including Valve's catalog.

SDL is a C library.  So how does it address this problem?

Let's look at its [API](https://wiki.libsdl.org/CategoryAPI).

### FreeImage

[FreeImage](http://freeimage.sourceforge.net/) is a C library that can decode various image formats (e.g. PNG, JPEG, GIF, and many others).  These images are compressed using complex algorithms so that they use less space on disk or less network bandwidth when transferred across a network.  If we want to show one of these images in a game for example we would need to be able to decode the image into an uncompressed format (something like BMP).  FreeImage can do this decoding for a large number of popular image formats.

Let's look at its [API](http://downloads.sourceforge.net/freeimage/FreeImage3160.pdf)

----

Notice any patterns?  All of these libraries prefix *everything* with the name of the library.  They do this in an attempt to avoid name collisions with other libraries.  While this mostly works it is a little awkward--everywhere we use a function from the library we have to use the prefix.  This makes the code somewhat more difficult to read and write.

The problem really is that in C, all names are in the same 'space'.  They all exist in a single global namespace.

C++ is supposed to be a better C.  So what language feature did the designers add to improve the name composition awkwardness we find in C?

## Namespaces

Namespaces are mechanism to partition the global namespace into named namespaces.  Each namespace provides its own scope.  Names from one scope could match names in another scope without conflict.

What kinds of names are we talking about?
- type names (classes, structs)
- enum names
- function names
- ...

We create namespaces by using the C++ keyword **namespace**.  

### Non-Programming Namespaces

We already know about the idea of namespaces from geography.  Consider the name Watertown.  If our namespace is the US Watertown could refer to several different places:

- Watertown, MA
- Watertown, CT
- Watertown, NY
- Watertown, TN
- Watertown, WI
- Watertown, MN
- Watertown, SD

So in the space of US town names Watertown is ambiguous.  But in the space of SD it is a specific place.  If you're in any of the above states it is likely that simply referring to Watertown would implicitly mean the city named Watertown in that state for local residents.  So when you're in SD you don't need to say Watertown, SD--Watertown is sufficient.

## Example

Consider the following:

```c++
namespace KPImage 
{
	class Bitmap { /* ... */ };

	// ...
}
```

Namespaces typically are used to group together logically related functionality.  For example a library used for image manipulation might use a namespace to group the various names within the library.

Within a namespace symbols from the same namespace can be accessed as they would if no namespaces were involved.  For example:

```c++
// KPImage.hpp
#pragma once

namespace KPImage 
{
	class Bitmap {
	public:
		int bitsPerPixel() const;
		// ...
	};

	Bitmap* createFromFile(string const &filename);
}
```

The createFromFile function is able to use the Bitmap class without qualification because they are both within the KPImage namespace.

## Qualification

Outside the namespace, names inside the namespace cannot be referred to without some extra effort.

Just like how classes have members, we also refer to the entities within a namespace as members of the namespace.  With classes we use the scope resolution operator :: to qualify class members.  For namespaces we also use the scope resolution operator in a similar way:

```c++
KPImage::Bitmap* bmp = KPImage::createFromFile("Tsoukalos.png");
// ...
delete bmp;
```

Since Bitmap and createFromFile are defined within KPNode, it would be an error to attempt to access them without qualification:

```c++
Bitmap* bmp = createFromFile("Tsoukalos.png"); // ERROR: 
```

What if we want to define createFromFile?  We could do this a couple ways:

```c++
#include "KPImage.hpp"

namespace KPImage  // re-open namespace
{
	Bitmap* createFromFile(string const &filename)
	{
		// TODO: actually implement
		return nullptr;
	}
}
```

Alternatively:

```c++
#include "KPImage.hpp"

KPImage::Bitmap* KPImage::createFromFile(string const &filename)
{
	// TODO: actually implement
	return nullptr;
}
```

I almost exclusively see the first style in code that I've seen.  The second style is better in that it can avoid accidental typos, but it really gets out of hand quickly.  Consider the implementation for the bitsPerPixel method on the Bitmap class within the KPImage namespace:

```c++
#include "KPImage.hpp"

int KPImage::Bitmap::bitsPerPixel() const
{
	return 32;
}
```

We have to constantly prefix every member definition with **KPImage::Bitmap::**.

## using-Directives

If we use a single type within a namespace many times it can be tiresome to re-type the namespace prefix again and again:

```c++
#include <iostream>
#include <string>

int main()
{
	std::string a = "foo";
	std::string b = "bar";

	std::cout << a << b << std::endl;

	return 0;
}
```

We need that qualification because string, cout, and endl are part of the C++ Standard Library and types within the Standard Library exist within the namespace std.

Instead we can use a using-directive.  A using-directive specifies that we would like every name from within a given namespace to be accessible without qualification.  For example:

```c++
#include <iostream>
#include <string>

using namespace std;

int main()
{
	string a = "foo";
	string b = "bar";

	cout << a << b << endl;

	return 0;
}
```

This makes code a lot easier to read and write.  You've probably been doing this for a while, but perhaps you didn't know *why* you were doing it.  using namespace std, just means that you would like to have all types within the std namespace to be accessible without qualification.

using-directives are not limited to the Standard Library.  We can use them for our own types as well.  Consider:

```c++
using namespace KPImage;

Bitmap* bmp = createFromFile("Tsoukalos.png");
// ...
delete bmp;
```

Notice that we can avoid explicitly qualifying the KPImage namespace for Bitmap and createFromFile due to the using-directive.

If we have using-directives for two or more namespaces that have conflicting names, then we must fully qualify only those names that conflict.  For example:

```c++
namespace A
{
	int x;
	void foo() {}
}

namespace B
{
	int x;
	void bar() {}
}

using namespace A;
using namespace B;

int main()
{
	foo();  // no conflict
	bar();  // no conflict

	x = 99; // ERROR: which x??

	A::x = 90; // OK
	B::x = 42; // OK

	return 0;
}
```

### Non-Global using-Directive

In the previous example we placed the using-directives at the global (file) scope.  This causes the directive to apply everywhere in that file after that directive.

If we want we can also place using-directives inside a local scope.  This causes the directive to only apply within that scope.  For example:

```c++
#include <iostream>
#include <string>

void bar();

int main()
{
	using namespace std;

	string a = "foo";

	cout << a;
	bar();

	return 0;
}

void bar()
{
	std::string b = "bar";

	std::cout << b << std::endl;
}
```

### Caution

We need to be careful with using-directives because if they're used carelessly they can partially re-introduce the problems that namespaces were intended to prevent.

Namespaces are still preferable because we can always reference the names directly when we need to.

Because of the way the C/C++ compilation model works--remember each source file is compiled independently--most name clashes that could happen might not unless two conflicting headers are get included (directly or indirectly) into the same source file.


## Using Directives in Headers

**Never place a using-directive in the global scope within a header file.**  You never know where a header file is going to end up getting included.  Once an offending header gets included somewhere you'll quickly end up with a disaster of conflicts.  I've unknowingly done this in the past and it caused huge problems.

Not being able to use global using-directives within a header file is extremely annoying.  I don't know of any good solution to this problem other than limit the number of namespaces you use and avoid placing external library types in your headers (a good idea anyway when possible).

This is a serious limitation and annoyance with namespaces in C++.

## using-Declarations

Instead of a using-directive, which makes all types within a namespace accessible we can instead use a *using-declaration*.  A using-declaration is like a using-directive, but it only applies to a single type.  Here's an example:

```c++
#include <iostream>
using std::cout; // cout means std::cout
```

Using-declarations aren't very common in my experience.  Usually when you're using a library you need to use a lot of different names from the library's namespace.


## Nested Namespaces

As with classes, namespaces are allowed to nest.  This can be useful to allow for multiple levels of namespaces.

```c++
namespace google
{
	namespace serialization
	{
		namespace protocolbuffers
		{
			void f();
			void g();
		}
	}
}

void google::serialization::protocolbuffers::f()
{
}
```

Nested namespaces are fairly common.  They exist in the C++ Standard Library as well as some popular C++ libraries.

## Namespace Aliases

Even namespaces themselves can and do clash, but the longer we make our namespace the less likely it would be for our name to conflict.

On the other hand dealing with long namespace names can be impractical.  This leads us to want shorter names.  So we have two design goals that lead to conflicting approaches.

One compromise is to use a longer namespace name and then use a namespace alias to provide a shorter name than you'll use in your code to refer to the longer namespace.

```c++
namespace pb = google::serialization::protocolbuffers;

void pb::g()
{
}
```

Like using directives, namespace aliases should never be used in header files.

## Explicit Global Scope

Occasionally we may need to reference the global scope explicitly:

```c++
namespace A
{
	int x; // charlie
}

namespace B
{
	namespace A
	{
		int x; // fredie
	}

	int foo()
	{
		return 
			A::x +  // fredie
			::A::x; // charlie
	}
}
```

This is normally pretty rare, but it does sometimes occur.  We can use the :: prefix anywhere to refer to the global scope--normally this is unneccessary.

## Namespaces in Applications

Namespaces are a great feature of C++.  If you're using C++ libraries that use namespaces you can more or less avoid most of the problems of name collisions and excessive name prefixing.

If you're writing a C++ library that will be integrated into programs developed independently by unrelated organizations then I would definitely recommend using namespaces.

Most programmers are not writing libraries that will be used by another organization.  Most developers are writing applications that integrate libraries written by another organization.  Does it make sense to use namespaces within such an application?

It is certainly acceptable to do so, however in my opinion the problems of namespaces within header files seriously compromises many of the advantages of using namespaces within C++.  In my experience namespaces are probably not worth using in C++ applications.  If you do use namespace, you probably would be best off to use a single namespace for the entire application.

For languages that do not use header files, I would have a stronger endorsement of namespaces for applications.  In these languages the import mechanism is typically much coarser (library level vs header file) so collisions are more likely.  Since they don't use headers the problems related to namespaces and headers don't exist.

## Namespaces Instead of Static Classes

A fairly common pattern is to group related functions into static functions on a class.  For example:

```c++
#include <string>
#include <vector>

class Base64
{
public:
	static std::vector<unsigned char> decode(std::string const &base64);
	static std::string encode(std::vector<unsigned char> const &buffer);
};
```

Sometimes a class with only static members is known as a *static class*.  Namespaces are a compelling alternative:

```c++
#include <string>
#include <vector>

namespace Base64
{
	std::vector<unsigned char> decode(std::string const &base64);
	std::string encode(std::vector<unsigned char> const &buffer);
};
```

We can use the namespace version almost identically to the class version, but the namespace version allows us to use using-declarations and using-directives if we wish.

If there were any static member variables, these can safely be moved into the .cpp file and use internal linkage.  This enables the same controlled access we can achieve with a class.