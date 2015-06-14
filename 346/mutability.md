# Mutability

Values in C++ can be either *mutable* (changeable, variable) or *immutable* (unchangeable, constant, const).  C++ contains very powerful and useful mechanisms for expressing mutability.

Systematic use of immutable objects has many advantages:

- code is easier to understand
- the compiler can detect more problems
- better performance

We've seen constants before:

```c++
int const a = 42;
a = 43;                  // ERROR: const variable cannot be assigned

int const b;             // ERROR: const object must be initialized

int const *c = &a;       // c points to a const int
c = c + 1;               // we can change where c points to
                         // but not the value that it points to
*c = 43;                 // ERROR: const variable cannot be assigned

                         // we can't change d or the value d points to
int const *const d = &a; // d is a const pointer to a const int
d = d + 1;               // ERROR: const variable cannot be assigned
*d = 43;                 // ERROR: const variable cannot be assigned
```

TIP: Read the statement backward and it flows like English.

## const Methods

C++ allows const-ness to be expressed on user-defined types as well.  To do this we define methods that can operate on const.  We can think of every C++ object as having a const interface and a non-const interface.  Non-const objects can access both the const interface and the non-const interface, but const objects can only access the const interface.  The const interface expresses operations which do not change the value of the object.

For our Date class let's define some const methods.  Perhaps we want to add a method that allows the caller to access the current year.  Since this operation does not change the state of our Date, it should be const.

```c++
class Date {
private:
	// ...
	int year;
public:
	// ...
	int getYear() const;
	void addYear(int yearCount);
};

int Date::getYear() const {
	return year;
}

void Date::addYear(int yearCount) {
	year += yearCount;
}
```

const is required as a suffix on both the declaration and defintion of getYear().

## const Checking

The compiler will check that we don't modify any member variables from within a const method.  If we do so my mistake we'll get a compiler error:

```c++
int Date::getYear() const {
	++year; // ERROR: 'year' cannot be modified in const method
	return year;
}
```

This also means that within a const method we can only call const methods of user-defined member variables.  For example:

```c++
class CalendarSystem {
public:
	void setName(char const *const name);
};

class Date {
private:
	// ...
	int year;
	CalendarSystem calendarSystem;
public:
	// ...
	int getYear() const;
	// ...
};

int Date::getYear() const {
	calendarSystem.setName("Gregorian"); // ERROR: cannot call non-const method
	return year;
}
```

This follows logically.  Const methods are not allowed to change member variables.  If the member variable is of a user-defined class, then calling a non-const method on that member could possibly change the value of that member, so it is not allowed.

Now let's make some instances of Date.  Remember that const Date instances can only call const methods.

```c++
Date const epoch(1, 1, 1970);
int const year = epoch.getYear(); // fine, const method
epoch.addYear(1); // ERROR: cannot call non-const method from const Date

Date birthday(12, 28, 2014);
birthday.addYear(1); // fine, birthday is non-const
int const year2 = birthday.getYear();
```

## const Method Override

const is part of the signature of a method.  This means that you can define both const and non-const overloads of a method.  Consider the following:

```c++
class Date {
private:
	int day;
	int month;
	int year;
public:
	Date(int month, int day, int year);

	void addYear(int yearCount);
	Date addYear(int yearCount) const;
};

void Date::addYear(int yearCount) {
	year += yearCount;
}

Date Date::addYear(int yearCount) const {
	return Date(month, day, year + yearCount);
}
```

Now Date has both a const and non-const overload for addYear().  The const version does not alter the current year, but instead returns a new Date that is yearCount years ahead of the current Date.

```c++
Date const epoch(1, 1, 1970);
Date birthday(12, 28, 2014);

// calls Date Date::addYear(int yearCount) const
Date const oneYearAfterEpoch = epoch.addYear(1); // fine, does not modify epoch

// calls void Date::addYear(int yearCount)
birthday.addYear(1); // fine, birthday is non-const
```

## const_cast

const_cast can be used to remove constness from a reference or pointer.  This works for both primative and user-defined types.  These types of operations should be avoided, because they circumvent the compiler error checking.

```c++
int const a = 42;
int const *const b = &a;
int *const c = const_cast<int *>(b);
*c = 43;
```

Occasionally a const_cast is needed for compatibility with C-based libraries, but its use should be avoided unless absolutely neccessary.  The whole point of const is to give the compiler additional information that allows it to find more errors for us.  Using const_cast effectively subverts the compilers error checking mechanisms.

The most grotesque cast of them all, C-style cast can also remove constness with ease.

```c++
int const a = 42;
*((int *)&a) = 43;
```

This cast is too powerful to be used safely.  It could easily remove constness without the programmer even realizing that it happened.  Don't use it in C++ programs.

## Logical Constness

Sometimes we need to represent something that is logically constant, but not physically constant.  This problem frequently happens with caching.  Let's say we have an accessor method for a relatively expensive operation.  We might want to store (cache) the value so that we don't need to recompute it every time.  The general term for this kind of operation is *lazy evaluation*.

Let's assume we have an operation called iso8601() that is expensive.
> Before incurring added complexity of this approach we should be sure that the operation actually *is* expensive.

```c++
class Date {
private:
	// ...

	int year;

	bool isCacheValid;
	string cache;

	string createIso8601() const;

public:
	// ...
	void addYear(int yearCount);
	// ...

	string iso8601() const;
};

string Date::iso8601() const {
	if (!isCacheValid) {
		cache = createIso8601(); // ERROR: modifying member in const method
		isCacheValid = true;     // ERROR: modifying member in const method
	}

	return cache;
}

void Date::addYear(int yearCount) {
	isCacheValid = false;
	year += yearCount;
}
```

Here iso8601() is logically or externally const.  Calling it does not change Date in any externally observable way.  We use the cache member variable to store the last string conversion so that if iso8601() is called a second time without any modification, we can return the previously calculated value without recomputing.

The problem is that we're attempting to change member variables within a const method, which is disallowed by the compiler.

There is a workaround however.  We can mark the member variables as mutable.  This allows them to be modified within const methods.

```c++
class Date {
private:
	// ...

	int year;

	mutable bool isCacheValid;
	mutable string cache;

	string createIso8601() const;

public:
	// ...
	void addYear(int yearCount);
	// ...

	string iso8601() const;
};

string Date::iso8601() const {
	if (!isCacheValid) {
		cache = createIso8601();
		isCacheValid = true;
	}

	return cache;
}

void Date::addYear(int yearCount) {
	isCacheValid = false;
	year += yearCount;
}
```

Mutable should only be used for cases of logical constness.  It should not be used as a general mechanism to subvert the type system of the compiler.

## Indirection

const methods are not allowed to modify the value of member variables, but they can change the values of objects they have a pointer to.  This might seem like a way of subverting the const mechanism, but really it isn't.  The address (where the pointer is pointing) of the pointer cannot be changed because it is a member.  The object pointed to is not a member, so its value can be changed.  Although we might think of this as a subpart of containing class, the compiler has no means of knowing that.  We could use this to rewrite the previous example:

```c++
struct Cache {
	bool isValid;
	string iso8601;
};

class Date {
private:
	// ...
	int year;
	Cache *cache;

	string createIso8601() const;

public:
	Date();
	void addYear(int yearCount);
	// ...

	string iso8601() const;
};

Date::Date()
{
	// TODO: memory leak!!  We'll learn how to fix this soon
	cache = new Cache;
}

string Date::iso8601() const {
	if (!cache->isValid) {
		cache->iso8601 = createIso8601();
		cache->isValid = true;
	}

	return cache;
}

void Date::addYear(int yearCount) {
	cache->isValid = false;
	year += yearCount;
}

string Date::createIso8601() const {
	// expensive operation
	// ...
}
```

## IMHO

const is one of the best features of C++ and one that is frequently overlooked or ignored.  It can really help improve the readability of your programs and find errors.

I remember one time where a coworker had been struggling to find the cause of a strange crash.  We added const to a few variables and suddenly the compiler told us exactly where the problem was.  The issue was an uninitialized variable.  When variables are declared const the compiler requires that they are initialized, allowing us to immediately find the source of the problem.

In my opinion all variables should be declared const unless they need to be changed.  I personally type const almost automatically when defining a variable, and then remove const only after I know I'm going to need to change the variable.  I would prefer if const was the default (or was at least as easy to type as non-const), but that would cause too many incompatibilities with existing code.