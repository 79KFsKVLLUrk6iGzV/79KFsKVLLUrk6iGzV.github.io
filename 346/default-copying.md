# Default Copying

By default objects support copying.  The default way of copying an object is to simply copy each of its members.  We can disallow copying or customize what happens during a copy, but we'll get to that later.

We already saw an example of copying:

```c++
Date sethMeyersBirthday = linusTorvaldsBirthday;
```

C++ supports several syntaxes for copying:

```c++
Date sethMeyersBirthday(linusTorvaldsBirthday);
```

and in C++ 11 we can do this (part of uniform initialization):


```c++
Date sethMeyersBirthday { linusTorvaldsBirthday };
```

or:

```c++
Date sethMeyersBirthday = { linusTorvaldsBirthday };
```

Note that this is the same syntax we use for primitive types:

```c++
int a = 42;
int b(42);
int d{42};    // C++ 11
int e = {42}; // C++ 11
```

We can also do assignment.  By default assignment does memberwise assignment, but this too can be disabled or overridden.

```c++
Date snlParty;
snlParty = sethMeyersBirthday;
```