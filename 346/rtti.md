---
title: Run-Time Type Information
---
# Run-Time Type Information

Given a hierarchy of classes such as the following:

![](dog-taxonomy.png)

We may need to perform conversions between types within the hierarchy.  There are two types of conversions that we might need to perform.

## Upcast

The first type of conversion is known as an *upcast*.  Given a pointer or reference to a subclass we can assign that to a pointer or reference respectively of the superclass.

This shouldn't be surprising.  Given a pointer to a Dog, we should be able to assign that to a pointer to any superclass (Canine, Carnivore, Mammal, Animal, Organism).  This follows from our general understanding of classification.  If you tell me that you drive a Camry I know that you have a car.

```c++
#include <iostream>
using namespace std;

class Animal {
public:
	virtual ~Animal() {}
	virtual void move() = 0;
};

class Dog : public Animal {
public:
	Dog() {}
	void move() override {
		cout << "moving Dog" << endl;
	}
};

int main()
{
	Dog *dog = new Dog();

	Animal *animal = dog; // upcast

	return 0;
}
```

Upcasting isn't a very interesting case.  We've seen it before.

## Downcast

Alternatively we may have a pointer or reference to a base class and wish to convert it to a derived type.  This is known as a *downcast*.

Unlike upcasting, downcasting isn't always going to work.  If you tell me that you have a vehicle, I can't assume you have a car.  Maybe you have an airplane or motorcycle.

Just as you can't assume something specific when you only know generalities neither can the compiler.  In our example if given a pointer to an Animal, the compiler cannot know if that Animal is a Dog or something else.

Downcasting is fairly common and certainly not wrong, but it is usually considered a 'code smell'.  That is, it is something we typically would prefer to avoid when we can by using a better design.  If you find yourself using a lot of downcasting in your program you might want to reconsider your design.

## dynamic_cast

To provide for downcasting C++ provides a special cast operator known as dynamic_cast.  We've seen several casts already (static_cast, reinterpret_cast, const_cast), but with dynamic_cast we now know of all the cast operators defined by the C++ language (C++ libraries can and do sometimes define additional cast operators).

Let's see an example:

```c++
#include <iostream>
using namespace std;

class Animal {
public:
	virtual ~Animal() {}
	virtual void move() = 0;
};

class Dog : public Animal {
public:
	Dog() {}
	void move() override {
		cout << "moving Dog" << endl;
	}
};

int main()
{
	Animal *animal = new Dog();

	Dog *dog = dynamic_cast<Dog *>(animal);
	cout << "address of dog: " << dog << endl;

	return 0;
}
```

Here the dynamic_cast says that we want to take animal (a pointer to an Animal) and convert it to a pointer to a Dog.  In this case it is pretty obvious that this will work because we just made it in the previous statement, but generally the compiler can't assume that this will work at runtime.

So what if dynamic_cast doesn't work?  Let's see an example:

```c++
#include <iostream>
using namespace std;

class Animal {
public:
	virtual ~Animal() {}
	virtual void move() = 0;
};

class Dog : public Animal {
public:
	Dog() {}
	void move() override {
		cout << "moving Dog" << endl;
	}
};

class Cat : public Animal {
public:
	Cat() {}
	void move() override {
		cout << "moving Cat" << endl;
	}
};

int main()
{
	Animal *animal = new Cat();

	Dog *dog = dynamic_cast<Dog *>(animal);

	cout << "address of dog: " << dog << endl;

	return 0;
}
```

In this case animal does not point to a Dog.  What does dynamic_cast do?  It returns nullptr.  If the conversion works we get back a pointer to the requested type, if it doesn't work we get back nullptr.

We should always check that the value returned by dynamic_cast isn't nullptr before we try to use it.  If we try to use a null pointer our program will crash.

These examples are really simple.  We are a little smarter than the compiler and we know when our conversions should work and when the shouldn't.  In this case we could also use static_cast.

static_cast and dynamic_cast can both be used for downcasting.  The difference is that static_cast will always return something for conversions between related types, whereas dynamic_cast will return nullptr if the conversion is invalid.  This makes dynamic_cast safer and more general for downcasting than static_cast.

dynamic_cast can also be used for upcasting, but it is not required.

dynamic_cast can only work with polymorphic types.  A type is polymorphic if it contains at least one virtual method.  Keep in mind that the virtual method could be in a base class.  dynamic_cast requires that the source object be of a polymorphic type.

Let's consider a more practical example.  Downcasting is often used when dealing with heterogenious collections.  Most of the collections we've seen so far have been homogenious, that is every element was of the same type.  It can also be useful to make collections where the elements types may not be the same.

Consider:

```c++
#include <iostream>
using namespace std;

class Animal {
public:
	virtual ~Animal() {}
	virtual void move() = 0;
	virtual void talk() = 0;
};

class Dog : public Animal {
public:
	Dog() {}
	void move() override {
		cout << "moving Dog" << endl;
	}
	void talk() override { cout << "woof woof" << endl; }
};

class Cat : public Animal {
public:
	Cat() {}
	void move() override {
		cout << "moving Cat" << endl;
	}
	void talk() override { cout << "meow meow" << endl; }
	void retractClaws() { cout << "retracting claws" << endl; }
};

int main()
{
	Animal *animals[] = {
		new Dog(),
		new Cat(),
		new Cat()
	};

	// make the animals move
	for (auto animal : animals)
		animal->move();

	// only have the dogs talk
	for (auto animal : animals)
	{
		Dog *dog = dynamic_cast<Dog*>(animal);
		if (dog != nullptr)
			dog->talk();
	}

	// make the cats retract their claws
	for (auto animal : animals)
	{
		Cat *cat = dynamic_cast<Cat*>(animal);
		if (cat != nullptr)
			cat->retractClaws();
	}

	// cleanup memory
	for (auto animal : animals)
		delete animal;

	return 0;
}
```

Heterogenious collections can be helpful when want to manage a single group of objects that are usually treated the same way.  Occationally we might still need to do something specific for one type, only cats support retracting their claws for example.

## Implementation

How does dynamic_cast actually work?  The whole point of dynamic_cast is to perform conversions for which there is not enough information available at compile time to determine if the conversion is safe.

To make this work, the compiler needs store additional data in the program for you.  Each polymorphic class has a virtual table (we saw that already).  To support dynamic_cast the compiler adds a field to the virtual table that points to a type information (type_info) object.  This information is added by the compiler at compile time.

When the compiler generates code for a dynamic_cast it uses that runtime information to determine if the conversion should be allowed given the runtime type of the object.  It uses the object's pointer to its class's virtual table to access the type_info object which says the runtime type of the object.  If the conversion makes sense given the runtime type information then the conversion is allowed to happen, otherwise dynamic_cast return nullptr.

![](rtti-impl.png)

We could implement this sort of behavior manually if we wanted, but since the language provides the implementation for us we don't need to.

Understanding how dynamic_cast is implemented it should be more obvious why the source object for dynamic_cast operations must be polymorphic.  If the object has no virtual methods then it will not have a virtual table.  Since the type information required for dynamic_cast is also stored in the vtable, we cannot use dynamic_cast on non-polymorphic types.

This also makes sense from a logical perspective.  Concrete types cannot be used without knowing their exact types.  Only polymorphic types can be safely manipulated via a base class pointer or reference.

Since the run-time type information (RTTI) required to implement dynamic_cast does impose a (very slight) size cost on the size of our programs there is usually an option to disable RTTI support in the compiler settings.  Compilers normally have RTTI enabled by default.

## dynamic_cast and References

Polymorphic behavior requires that objects be manipulated via a pointer or reference.  In the previous examples of downcasts using dynamic_cast we used pointers, but we might also want to use references.  Consider the following:

```c++
Dog dog;

Animal &animal = dog;
animal.talk();

dynamic_cast<Dog&>(animal).talk();
dynamic_cast<Cat&>(animal).retractClaws();
```

This brings up a question however, what if the conversion fails (as in the second dynamic_cast)?  Remember that in C++ references cannot be null.  This means that returning nullptr isn't a valid option for dynamic_cast when the target type is a reference.

What happens is dynamic_cast throws a bad_cast exception.  We haven't covered exceptions yet, but suffice it to say that your program will crash.  

## static_cast vs dynamic_cast

As we saw earlier, we can use either static_cast or dynamic_cast for downcasting (as well as upcasting, but it isn't neccessary in that case).  The main difference between these operators (when used for downcasting) is whether or not the operations are checked or unchecked.

dynamic_cast is a safer operation because it is checked at run-time to ensure that the operation is safe.  static_cast on the other hand does not check for correctness.  Consider the following example:

```c++
Cat c;
Animal *catPtr = &c;

Dog *dog = static_cast<Dog*>(catPtr);
```

This static_cast is invalid and should not be allowed, but the compiler will allow it.  The result is undefined.  It might work fine, not work at all, or cause some weird problem at run-time.

Since dynamic_cast checked it will not allow invalid conversions.  Attempt to perform such operations will either result in nullptr (for pointers) or an exception (for references).  Basically the result of the operation is well-known and always defined since it is checked at run-time.

Since dynamic_cast is the safer cast to use for downcasting should static_cast ever be used, or should we just always use static_cast for downcasting?  There are a few reasons why you might find or write code that uses static_cast for downcasting.

The first reason is that since static_cast does not do the run-time checking that dynamic_cast does it is slightly faster.  dynamic_cast is also very fast, but it cannot be as fast as a static_cast (for downcasting).  In an extreme performance situation we might choose to use static_cast for performance, although if we really have such tight performance requirements we might want to consider avoiding polymorphic types all together.

Another reason to see static_cast used for downcasting is that dynamic_cast is a newer mechanism.  There was a lot of C++ code written before the language or compilers supported dynamic_cast.  This code would typically do something like what was shown in the implementation section manually instead of letting the compiler do it for us.

## typeid

In the implementation section a careful observer would have noticed a type_info type that described the type of a polymorphic object.  This isn't just an internal implemenation detail of the compiler.  This is an actual type that is found in the C++ standard library header \<typeinfo\>.

We can get the type_info for an object by using the typeid operator.  This is similar to the sizeof operator in that it looks like a function, but it is actually part of the C++ language.

If the object passed to typeid() is non-polymorphic then the type_info for the object will be determined at compile-time.  If the object is polymorphic then the type_info will be determined using the same run-time information that is used to implement dynamic_cast.

Here's an example:

```c++
Cat c;
Animal *catPtr = &c;

auto const &type = typeid(*catPtr);
cout << type.name() << endl; // prints out something like Cat
```