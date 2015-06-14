# Operator Overloading

C++ supports something called *operator overloading*.  This provides a way to write user-defined types that support built-in operators (e.g. +, -, ++, ...).

## Example

Operator overloading is an important powerful feature of C++ that enables some really easy to use libraries.

Let's have a look at a class that you're probably already familiar with, the string class in the standard library (std::string).  This class is used to represent strings.  Let's look at the [API](http://www.cplusplus.com/reference/string/string/).

We can see that std::string supports several operators:

 - **operator=** <br> Used for string assignment (strcpy)
 - **operator[]** <br> Used to access a char at a particular index
 - **operator+=** <br> Used to concatenate another string at the end of the current string (strcat)
 - **operator+** <br> Used to concatenate two strings into a new string (strcat)
 - **operator==** and **operator!=** <br> Used to compare two strings for equality or non-equality (strcmp)
 - **operator<** and **operator<=** and **operator>** and **operator>=** <br> Used to compare two strings lexicographically (strcmp)


Let's see an example where we use these operators:

```c++
#include <string>
using namespace std;

int main()
{
	string name = "fred";
	name = "bob"; // operator= (assignment)

	char ch = name[0]; // operator[]

	string fullName = name + " frankenfoot"; // operator+

	bool eq = name == fullName; // operator==
	bool neq = name != fullName; // operator!=

	bool lt = name < fullName;
	bool gt = name > fullName;
	bool le = name <= fullName;
	bool ge = name >= fullName;

	return 0;
}
```

These overloaded string operators make our program easier to read and write.

## Mirror Universe

You might be thinking, "Big deal, that's something build-in to C++."  But here's the thing--as far as the C++ language or compiler is concerned std::string isn't a special type.  It is no different than your own types.

Some languages do not provide for operator-overloading.  In an alternate reality mirror universe where C++ didn't support operator overloading our previous code might have look like this (I'm just making up names here, none of this will actually work).

```c++
#include <string>
using namespace std;

int main()
{
	string name = "fred";
	name.assign("bob");

	char ch = name.at(0);

	string fullName = string::concat(name, " frankenfoot");

	bool eq = string::compare(name, fullName);
	bool neq = !string::compare(name, fullName);

	bool lt = string::compare(name, fullName) < 0;
	bool gt = string::compare(name, fullName) > 0;
	bool le = string::compare(name, fullName) <= 0;
	bool ge = string::compare(name, fullName) >= 0;

	return 0;
}
```

Which is better?  Operator overloading makes the code a little easier to read/write.  Without it the code is perhaps more clear as to what is actually happening.

## Controversies

Many languages have been made since C++, and operator overloading seems to be somewhat controversial among language designers.  Java came after C++, but dropped operator overloading.  C# came after Java, but C# supports operator overloading.

Here's a quote from James Gosling, the 'father of Java':
> There are some things that I kind of feel torn about, like operator overloading. I left out operator overloading as a fairly personal choice because I had seen too many people abuse it in C++. I've spent a lot of time in the past five to six years surveying people about operator overloading and it's really fascinating, because you get the community broken into three pieces: Probably about 20 to 30 percent of the population think of operator overloading as the spawn of the devil; somebody has done something with operator overloading that has just really ticked them off, because they've used like + for list insertion and it makes life really, really confusing. A lot of that problem stems from the fact that there are only about half a dozen operators you can sensibly overload, and yet there are thousands or millions of operators that people would like to define -- so you have to pick, and often the choices conflict with your sense of intuition. Then there's a community of about 10 percent that have actually used operator overloading appropriately and who really care about it, and for whom it's actually really important; this is almost exclusively people who do numerical work, where the notation is very important to appealing to people's intuition, because they come into it with an intuition about what the + means, and the ability to say "a + b" where a and b are complex numbers or matrices or something really does make sense. You get kind of shaky when you get to things like multiply because there are actually multiple kinds of multiplication operators -- there's vector product, and dot product, which are fundamentally very different. And yet there's only one operator, so what do you do? And there's no operator for square-root. Those two camps are the poles, and then there's this mush in the middle of 60-odd percent who really couldn't care much either way. The camp of people that think that operator overloading is a bad idea has been, simply from my informal statistical sampling, significantly larger and certainly more vocal than the numerical guys. So, given the way that things have gone today where some features in the language are voted on by the community -- it's not just like some little standards committee, it really is large-scale -- it would be pretty hard to get operator overloading in. And yet it leaves this one community of fairly important folks kind of totally shut out. It's a flavor of the tragedy of the commons problem.

What does he mean about operator overloading being potentially confusing?  Here's a terrible example of operator overloading:

```c++
#include <iostream>
using namespace std;

class Foo
{
public:
	void operator+=(char ch) {
		cout << ch;
	}
};

int main()
{
	Foo f;
	f += 'F';
	f += 'A';
	f += 'I';
	f += 'L';

	return 0;
}
```

This shows why some people hate operator overloading.  I've just defined the += operator on Foo to mean "print a char to the console".  This is a pretty unreasonable action for +=.

Some people say that this shows you shouldn't have operator overloading.  Other people say that you can use regular methods in ways that are just as bad.  Neither side is really wrong.

## Opinion

So is operator overloading good or evil or somewhere in between?

I actually think it is a good thing, but with some caveats.  I think that it is nice to be able to use built-in operators on user-defined types.  It is nice to be able to have classes that are easy to use as built-in types (such as std::string).

The big caveat is that types where operator overloading really makes sense tend to be *very* low-level or primitive types.  Something like the std::string class.

Here's a few more examples of good uses of operator overloading:

- [complex](http://www.cplusplus.com/reference/complex/complex/) <br>The complex type from the C++ Standard Library represents a complex number.  It supports various [operators](http://www.cplusplus.com/reference/complex/complex/operators/) that you would expect for a numerical type.
- [duration](http://www.cplusplus.com/reference/chrono/duration/) <br>The duration type from the C++ Standard Library represents a period of time.  It supports various [operators](http://www.cplusplus.com/reference/chrono/duration/operators/) to represent things like adding one duration to another duration.
- [shared_ptr](http://www.cplusplus.com/reference/memory/shared_ptr/) <br>Uses operator overloading to look like a native pointer, but can automatically manage object lifetimes.


Here's the other part of my caveat:  When you're doing something so basic or primitive that operator overloading might make sense there is probably already some existing library that you could use instead of writing your own.

Here's my overall recommendation if you think you might want to use operator overloading:

1. Make sure that the operations make sense given the meaning of the same operators for primitive types.  If they don't, stop here.  Don't use operator overloading for your type.
2. Make sure that there isn't already a type that does the same thing as your type built into the C++ Standard Library or some other C++ library.  Since types where operator overloading make sense tend to be very primitive, they tend to be needed in a lot of different programs so somebody probably has already made a really good library that does the same thing.  Try to use their library before trying to build your own.  If that type works stop here.
3. If you've gotten to this point study up more on how to correctly do operator overloading and then implement your type.

I'm not going to spend a lot of time going into all the details of how to implement operator overloading.  In my experience actually writing code that implements operator overloading is very rare (as opposed to using operator overloads which is very common).

I think our time could be better served by focusing on more commonly used features. 