# Properties

A common pattern in OOP is to expose data member fields through methods.  We sometimes refer to these as properties.  Many OOP languages have direct language support for properties, but C++ does not.  Properties are still a useful way of thinking about and modeling classes even in C++.

## Basic Property

As an example let's make an object with a basic property.  Let's make a shape class that has a color property:

```c++
enum Color { Red, Green };

class Shape {
private:
	Color _color;
public:
	Color color() const {
		return _color;
	}
	void setColor(Color color) {
		_color = color;
	}
};
```

Notice that this property is composed of two methods.

Shape::color returns the current color.  This method is called an accessor, or *getter*.  Accessor methods normally should be const, since getting the value of a property should not normally change the object.

Shape::setColor is changes the current color value of the class.  This method is called a *setter* method.  A setter will normally not be declared const because it will change the state of the object.

Properties are normally public, but they may have occasionally be less accessible.

The field where the property's value is stored is known as a *backing field*.

## Naming

If we think of the Shape type as having a color property, then we could have use different names for the actual method names.  Since C++ has no direct support for the concept of a property within the language, the method names we choose are up to us.  That said, there are some commonly used names.

```c++
Color color() const
void setColor(Color color);

Color getColor() const
void setColor(Color color);

Color color() const;
Color& color();
```

I generally prefer the first style.  I feel like it leads to very readable code.

## Read-Only Properties

A property may have only a getter and no setter.  In this case the property is known as a read-only property.

```c++
enum Color { Red, Green };

class Shape {
private:
	Color const _color;
public:
	Shape(Color const color) : _color(color) {
	}
	Color color() const {
		return _color;
	}
};
```

In this example a Shape's color cannot be changed after the object is created.  In this case it would also make sense to make the backing field const.

Alternatively properties could be read-only because they can only be changed through some other mechanism.  In this case the backing field would most likely not be const.

## Write-Only Properties

Less often we may need a property that can only be saved and not read.

```c++
enum Color { Red, Green };

class Shape {
private:
	Color _color;
public:
	void setColor(Color color) {
		_color = color;
	}
};
```

## Computed Properties

We may have read-only properties which have no backing store.  In this case the property is known as a *computed property*.  For example:

```c++
class Person {
private:
	Date const _dateOfBirth;
public:
	int age() const {
		return (Date::today() - _dateOfBirth).totalYears();
	}
};
```

In this example (using a made up Date class) the Person's age is computed based on their date of birth.  Notice there is no backing field for the person's date of birth.

## Lazy Properties

A *lazy property* is property whose value is not computed until the first time the property is used.  Lazy properties are useful when the property needs to do something expensive and may not always need to be accessed.  Lazy properties are a special case of computed properties.

Consider this example:

```c++
struct Node {
	int value;
	Node *next;
};

class List {
private:
	Node * _head = nullptr;
	mutable Node * _tail = nullptr;

	Node* findTail() const {
		auto current = _head;
		while (current != nullptr) {
			auto const next = current->next;
			if (next == nullptr) {
				_tail = current;
				return _tail;
			}
			current = next;
		}
		return nullptr;
	}

public:
	void prepend(int const value) {
		_head = new Node { value, _head ? _head : nullptr };
	}

	Node const* tail() const {
		if (_tail != nullptr)
			return _tail;

		_tail = findTail();
		return _tail;
	}
};
```

Our list has a lazy property called tail that computes the tail.  If we don't currently know what the tail is, we search for it.  Once we've found the tail we save it.  Note that this example is a little silly because if we needed the tail it would probably be easier to just maintain it.

Normally in life we think of laziness and a negative characteristic, but in programming it can be a good way to optimize performance.  In fact the concept of laziness as done in programming is could be useful in life as well.  Laziness in programming would mean roughly this in real life--if you have a task that you probably won't need to do and it will take a long time to do, then wait to do it until you are actually are sure that you need to and once you've done it avoid doing it again.

The concept of laziness is so interesting an powerful that it is one of the principle motivators for many functional programming languages.  Frequently laziness can be done automatically in functional languages.  In C-based languages we need to implement laziness ourselves.

We can also use laziness to represent unbounded sequences.

## Purpose

If we have both a getter and setter for a property it might seem silly not to just expose the field directly, however this is probably not a good idea.

When we expose the field directly we cannot as easily know where our field is being used.  This makes it more difficult to change the type in the future because we have to track down all the places it was used.  If it was used by a 3rd party it probably cannot be changed.

When we expose the field directly we cannot easily change the underlying representation without breaking the users of the class.  We cannot change from a normal field to a lazy field.

Setter methods allow us to prohibit illegal or non-sensical values.  We can enforce class invariants.

Setter methods also allow classes to respond in some way to the set.  Consider the following:

```c++
enum Color { Red, Green };

class Shape {
private:
	Color _color;
public:
	Color color() const {
		return _color;
	}
	void setColor(Color color) {
		_color = color;
		redraw(); // color changed, need to redraw
	}
	// ..
};
```

When the Shape's color changes we need to redraw the the Shape.  If the _color field was exposed directly we would have no way of knowing when it was changed or when we should call redraw.