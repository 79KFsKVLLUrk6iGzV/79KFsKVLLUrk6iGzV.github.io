# Access Control

With our Date struct there are no restrictions on how it can be used externally.  We might want to specify an interface that defines how our type may be used.  This interface can hide the specifics of how our type is implemented.

With structs everything is implicitly *public* (externally accessible).  Take the following example:

```c++
Date linusTorvaldsBirthday;
linusTorvaldsBirthday.init(12, 28, 1969);

linusTorvaldsBirthday.day = 4000;
```

Clearly 4000 isn't a valid day.  Since day is public, code using Date is free to modify any of its members at will.  Perhaps instead we would want to design our Date class such that invalid data would be disallowed.

## private/public

There is an access modifier called *private* that makes class members hidden from outside the class.  Private members are still accessible from inside the type, but cannot be accessed externally.  We may make both methods and fields (member variables) private.

Another access modifier is called *public*.  When a member is public then it is accessible from outside the class.

Access specifiers are used to define sections that fall into a given access level.  In the example below we have two sections.  The first is private and the second is public.

Let's change Date to restrict access to its implementation details:

```c++
struct Date {
private: // unless I say otherwise, make everything below private
	int day;
	int month;
	int year;
public:  // unless I say otherwise, make everything below public
	void init(int month, int day, int year);
	void addYear(int yearCount);

	// ...
};
```

Now access to the private section is prohibited outside of Date's members.  The following code will now result in a compiler error:

```c++
Date linusTorvaldsBirthday;
linusTorvaldsBirthday.init(12, 28, 1969);


linusTorvaldsBirthday.day = 4000; // ERROR: cannot access private member
```

## class

Alternatively we could change Date from a struct to a *class*.  class is another keyword used to define custom types.  Like struct, class defines the blueprint for instances of that type.

```c++
class Date {
// implicitly private
	int day;
	int month;
	int year;
public:
	void init(int month, int day, int year);
	void addYear(int yearCount);

	// ...
}; 
```

The member variables (day, month, and year) are still private, because members are private by default within a class.

Even with a class we can (and probably should) be explicit about which parts are private.  By stating the access level explicitly we can avoid confusion and make the code more accessible.

```c++
class Date {
private: // explicitly private
	int day;
	int month;
	int year;
public:
	void init(int month, int day, int year);
	void addYear(int yearCount);

	// ...
};
```

A class interface is often used to express *invariants*, that is, things that cannot change (structs can too, but this is unconventional).  For our Date class an invariant might be that it represents a legal date.  For a Url class it might be that it follows the rules of the [Uri specification](https://tools.ietf.org/html/rfc3986).

The public/private sections may appear in any order.  There can be any number of public/private sections within a type.  Different people have different styles with regard to preferred ordering and section counts.

## Why Limit Access?

At this point you might be wondering what the point of restricting access could be.  Why not make everything public in case it does need to be accessed externally?

Advantages of separating interface (public) from implementation (private):

- Illegal values/usage can be restricted.
- Implementation changes can be isolated to one type (callers need not change).
- Interface serves as documentation on how to use a type.
- Design can be improved.
- When making changes, finding all the places something private is used is easy.  If everything is public, it is harder (or sometimes impossible) to change.

Both of these items are most important when you're working on a team with other people or writing/using a library made by/for someone else.  The basic idea is to design software components that can work as black boxes.  That is they can be used without fully understanding how they work.

Also remember that public/private in classes is not the only way to restrict access.  We can also use C mechanisms like file separation and linkage specifiers to control access.  These techniques are not mutually exclusive.  They can often be combined to great effect.

## Pointer Malfeasance

Private members can still be accessed via pointer tricks and other dirty mechanisms.  These mechanisms should be avoided.  Making a member private does not mean it is somehow impossible for malicious code to access or more secure.  Writing such code is a dirty trick, and should be avoided.

For the curious here is an example, but not a recommendation:

```c++
Date date;
date.init(1, 1, 1970);

// DON'T DO THIS!!!
*reinterpret_cast<int*>(&date) = 4000; // change date.day to 4000
```

## class vs struct

In C++ class and struct can really be used interchangeably.  They do the same thing, except for different default rules for access specifiers (public/private).

That said a typical convention is to use structs only in the case of simple data records where all members are often public and in which few if any invariants are enforced.

Classes are typically used for more complex types that express invariants.

Classes are used most of the time in my experience.