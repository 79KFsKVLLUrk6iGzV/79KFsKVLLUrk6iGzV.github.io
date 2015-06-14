---
title: Member Types
---
# Member Types

Classes can contain also contain types.  Member types are frequently called *nested* types.  So we might have a nested class or a nested enum.

You can actually nest a type within a nested type, so you can have as many levels of nesting as you desire.  It is pretty unusual to see nesting beyond 1 or 2 levels.

## Nested Enum

Here Date defines a nested enum:

```c++
class Date {
public:
	enum CalendarType {
		Gregorian,
		Islamic,
		Hindu
	};
};

Date::CalendarType const type = Date::Gregorian;
```

## Nested Class

Classes can also contain nested classes.  A nested class may access the private static variables of its containing class.

```c++
// A.hpp
#prama once

class A {
private:
	static int _foo;
	int _bar;
public:
	class B {
	public:
		int f();
		int b();
	};
};
```

```c++
// A.cpp
#include "A.hpp"

// notice the A::B::f (f is a function defined on B which is defined on A
int A::B::f() {
	return _foo; // legal
}

int A::B::b() {
	return _bar; // ERROR: what's _bar??
}
```

Nested classes can access private non-static variables but only if they have a reference to the parent.  Consider:

```c++
class Parent {
private:
	int _a;
public:
	explicit Parent(int a);

	class Child {
	private:
		Parent * _parent;
	public:
		explicit Child(Parent *parent);
		int getA();
	};

	Child * makeChild();
};

Parent::Parent(int a) {
	_a = a;
}

Parent::Child::Child(Parent *parent) {
	_parent = parent;
}

int Parent::Child::getA() {
	return _parent->_a;
}

Parent::Child* Parent::makeChild() {
	return new Child(this);
}


int main() {
	Parent parent(42);

	Parent::Child *child = parent.makeChild();
	int a = child->getA();

	delete child;

	return 0;
}
```