---
title: Derived Types
---
# Derived Types

We often build software systems with concepts that exist in the world around us.  A system like YouTube represents ideas like videos and users.  A system like WebAdvisor models concepts like courses, students, faculty.

These concepts are often related to each other some how.  For example, students and faculty are both people (seriously faculty are actually people), so they may share some characteristics in common.

The process of taking these concepts and implementing them in software systems is a process called *modeling*.  Modeling takes the characteristics of things in the world around us and simplifies them.  We ignoring details that are not pertinent to the system at hand and we make concepts more precise.

As an example imagine a computerized card game.  We can model things like a deck of cards.  But when we model the deck we probably ignore a huge number of real-world characteristics that are irrelevant to our game.  We probably ignore things like the thickness or the cards, the material of the card, the possibility that some cards might be lost, the fact that some cards could be bent, or the subatomic structure of the card materials.  All of those characteristics exist in real cards, but they probably aren't important to our card game so we exclude them from our model.

Modeling isn't just about characteristics of things.  We also might want to model how things are interrelated.  We've already seen a few ways we can achieve this modeling:
 - Classes and objects also provide an *instance-of* relationship--John is an instance of the class Person.  We use classes to model the characteristics that are common to all instances of a class.
 - Through member variables we can model a *has-a* relationship--a instance of Person has a name.

In addition to the previous ways of expressing relationships, we might want to model things through hierarchy or taxonomy.  Here we express how classes are interrelated.  My dog (Chloe) is an instance of the class Dog.  All Dogs Canines.  Canines are Carnivores.  Canines are Mammals.  Mammals are Animals.  Animals are Organisms.  We can draw this relationship as follows:

![](dog-taxonomy.png)

In C++ we would write the above diagram as follows:

```c++
class Organism {};
class Animal : public Organism {};
class Mammal : public Animal {};
class Carnivore {};
class Canine : public Carnivore, public Mammal {};
class Dog : public Canine {};

Dog chloe;
```

Just a bit more terminology and we'll be ready to continue.

We use the term *superclass* to describe a higher class in the hierarchy and *subclass* to describe a lower class.  So as an example Dog is a subclass of Canine.  Canine is a superclass of Dog.  Organism is also a superclass of Dog.  So are Canine, Carnivore, Mammal, and Animal.  Animal, Mammal, Carnivore, Canine, and Dog are all subclasses of Organism.  If you draw the is-a arrows going upward, then you can think of super as above and sub as below.

A superclass should describe all the characteristics that are common to all subclasses.  If we read about mammals, we'll learn that they have hair and mammary glands as well as other characteristics that are common for all mammals.  Since a canine is also a mammal we can think of it taking all the characteristics of mammals and then adding some additional characteristics that are unique to canines.

In OOP we call the process of taking (or inheriting) characteristics from your superclass as *inheritance*.  We say that a subclass is *derived* from its superclass.  So Dog would be derived from Canine.

Another thing to notice in the diagram above is that Canine derives from both Mammal and Carnivore.  This is known as *multiple inheritance* as opposed to the case of deriving from a single base class which is known as *single inheritance*.  We'll cover single inheritance before we look at multiple inheritance.

## Example

Let's imagine we're building a 2D top-down game.  In the game we have vehicles that can move around.  Each vehicle has a x and y coordinate.  We might have a stuct as follows:

```c++
struct Vehicle
{
	int x;
	int y;
};
```

We might also have airplanes in our game.  They also have x and y positions, but they also have a z position (how high they're flying).

```c++
struct Helicopter
{
	int x;
	int y;
	int z;
};
```

Let's imagine that we have some part of the game that allows one vehicle to follow another.  It would probably be more complex than this, but for an example let's start with the following:

```c++
void follow(Vehicle const &leader, Vehicle &follower, int distance)
{
	// not actually correct
	follower.x = leader.x + distance;
	follower.y = leader.y + distance;
}
```

Now it might be nice to be able to have a helicopter follow a car (police chase?), but it won't work:

```c++
Vehicle crimalCar{ 1, 2 };
Helicopter policeChopper;

follow(crimalCar, policeChopper, 1); // ERROR: cannot convert argument 2 from 'Helicopter' to 'Vehicle &'
```

This doesn't work even though it might seem like it should.  The problem is that the compiler has no idea that Vehicle and Helicopter are in any way related.

There's another more subtile problem.  We had to re-write a part of Vehicle inside of Helicopter.  In this example it is a pretty small amount of duplication, but in a more realistic example it would probably be more.  Regardless of amount we should try to eliminate code duplication whenever possible.

Let's fix both of these problems by re-writing Helicopter to derive from Vehicle:

```c++
struct Vehicle
{
	int x;
	int y;
};

struct Helicopter : public Vehicle
{
	int z;
};
```

At this point the follow function will work exactly the way we wanted and the duplication between Vehicle and Helicopter has been eliminated.  Notice that we say: **: public Vehicle** after the definition of Helicopter to say that Helicopter derives from Vehicle.

Even though x and y are no longer listed in Helicopter's definition they still exist as part of Helicopter by virtue of being derived from Vehicle.

We can use the sizeof operator and verify that indeed there is at least room for all 3 fields.

```c++
Vehicle crimalCar { 1, 2 };
Helicopter policeChopper;

auto const vehicleSize = sizeof(crimalCar);         // 8
auto const helicopterSize = sizeof(policeChopper);  // 12
```

This code also works now:

```c++
Vehicle crimalCar{ 1, 2 };
Helicopter policeChopper;

follow(crimalCar, policeChopper, 1);
```

## Subclass/Superclass Conversions

When accessed via a pointer or reference we can treat an instance of a derived class as though it were an instance of its base class.  This rule follows logically from the earlier section on dogs and animals.  All Dogs are Canines, but the opposite is not true (some Canines could be Wolves).

We already saw one example of this.  The policeChopper was treated as an instance of vehicle.  Let's see a few more:

```c++
Vehicle crimalCar{ 1, 2 };
Helicopter policeChopper;

policeChopper.z = 10;

follow(crimalCar, policeChopper, 1);

Vehicle *a = &policeChopper; // OK: Helicopters are Vehicles
Helicopter *b = &crimalCar; // ERROR: not all vehicles are helicopters

Helicopter *c = static_cast<Helicopter*>(a); // OK in this case

// will likely cause runtime error or memory corruption
Helicopter *d = static_cast<Helicopter*>(&crimalCar); // NOT OK (but compiles)
d->z = 9;
```

static_cast can be used to convert between pointers to related types.  This is basically a means of telling the compiler that you know that the conversion is safe in this case.  In the example above we're telling the compiler to allow us to convert a pointer to a Vehicle to a pointer to a Helicopter because in this case we know that particular instance is a Helicopter.  The compiler doesn't do any kind of run-time checking.  If we tell the compiler wrong (as we did later in that example) it will likely cause serious problems.

## Headers

The base class needs to be defined before we can derive from it.  This means we'll need to include the header files for the base types in the derived type's header file.  For example:

```c++
// Vehicle.hpp
#pragma once

struct Vehicle
{
	int x;
	int y;
};
```

```c++
// Helicopter.hpp
#pragma once
#include "Vehicle.hpp" // definition of base class needed 

struct Helicopter : public Vehicle
{
	int z;
};
```


## Constructors

Let's switch our types to classes and give them constructors.


```c++
class Vehicle
{
private:
	int _x;
	int _y;
public:
	Vehicle(int const x, int const y);
};

class Helicopter : public Vehicle
{
private:
	int _z;
public:
	Helicopter(int const x, int const y, int const z);
};

Vehicle::Vehicle(int const x, int const y) : _x(x), _y(y)
{
}

Helicopter::Helicopter(int const x, int const y, int const z) : Vehicle(x, y), _z(z)
{
}
```

Notice that in Helicopter's member initializer list we call Vehicle's constructor.

Base class constructors run before the derived class constructor runs.  In other words objects are created from base class to derived class.

## Constructor/Destructor Order

When we derive one type from another we need to be aware of the ordering of constructor and destructors.  Each class within the inheritance tree will have constructors/destructors.  C++ has very specific rules as far as the order of these operations.

Let's look at an example:

```c++
class First {
public:
	First() { cout << "First()" << endl; }
	~First() { cout << "~First()" << endl; }
};

class Second {
public:
	Second() { cout << "Second()" << endl; }
	~Second() { cout << "~Second()" << endl; }
};

class Base {
public:
	Base() { cout << "Base()" << endl; }
	~Base() { cout << "~Base()" << endl; }
};

class Derived : public Base {
private:
	First _first;
	Second _second;
public:
	Derived() { cout << "Derived()" << endl; }
	~Derived() { cout << "~Derived()" << endl; }
};

int main()
{
	Derived d;

	return 0;
}
```

The preceeding will output the following:

```
Base()
First()
Second()
Derived()
~Derived()
~Second()
~First()
~Base()
```

From this we can infer the general rules for how an object is constructed:

1. Base class constructor runs
2. Member variables are constructed (in the order they are declared in the class)
3. Body of the class constructor 

And for destruction (which is exactly the opposite order):

1. Body of the class destructor
2. Member variables are destroyed (in the opposite order from which they are declared)
3. Base class destructor runs

The construction and destruction happens in this order so that an object cannot be used before it is constructed or after it is destroyed.

It also might be a little surprising that member initialization happens in the order in which the members are listed in the class definition instead of the order in which the are called in the constructors member initializer list.  It happens this way so that the destruction can happen in the opposite order as initialization even if there are multiple constructors with different member initializer list order.  Without this rule, guarenteeing proper destruction order would require extra runtime overhead to remember which constructor overload had created the object.

## Methods

Let's give our classes a print method.  It is pretty common that we might want to have a method with the same signature in different related classes.  By giving the same names to operations in a class hierarchy we hope to be able to treat various classes in similar ways.

```c++
class Vehicle
{
private:
	int _x;
	int _y;
public:
	Vehicle(int const x, int const y);
	void print() const;
	void printPosition() const;
};

class Helicopter : public Vehicle
{
private:
	int _z;
public:
	Helicopter(int const x, int const y, int const z);
	void print() const;
	void printPosition() const;
};

Vehicle::Vehicle(int const x, int const y) : _x(x), _y(y)
{
}

Helicopter::Helicopter(int const x, int const y, int const z) : Vehicle(x, y), _z(z)
{
}

void Vehicle::printPosition() const
{
	cout <<  _x << ", " << _y;
}

void Helicopter::printPosition() const
{
	cout << _x << ", " << _y << ", " << _z; // ERROR: cannot access private _x, _y
}

void Vehicle::print() const
{
	cout << "Vehicle at (";
	printPosition();
	cout << ")" << endl;
}

void Helicopter::print() const
{
	cout << "Helicopter at (";
	printPosition();
	cout << ")" << endl;
}
```

Unfortunately this won't work.  The problem is that _x and _y are private.  Private variables cannot be accessed in derived classes.  This might seem surprising, but if this wasn't true then any any private member variable could be accessed by creating a subclass.  This would mean that we couldn't assume that private variables were only used within that class's methods.  Any change to private members would potentially require making changes in every subclass.  This might be possible (but frustrating) in our own programs, but when the class is in a shared library it is practically impossible.

There are actually a few ways approaches that can address this problem.

### Calling Base Methods

If there is a method with the same name in both the base class and the subclass with the same name, by default the method in the current class will be called.  We can override that behavior and explicitly request the base class version of the method by using the typename and scope resolution operator.

```c++
void Vehicle::printPosition() const
{
	cout <<  _x << ", " << _y;
}

void Helicopter::printPosition() const
{
	Vehicle::printPosition(); // call to base
	cout << ", " << _z;
}

void Vehicle::print() const
{
	cout << "Vehicle at (";
	printPosition();
	cout << ")" << endl;
}

void Helicopter::print() const
{
	cout << "Helicopter at (";
	printPosition();
	cout << ")" << endl;
}
```

Here Helicopter::printPosition explicitly calls Vehicle's version of printPosition.  If we would have left off the Vehicle:: from the call, then Helicopter::printPosition would have been called recursively, leading to infinite recursive and stack overflow.

```c++
void Helicopter::printPosition() const
{
	printPosition(); // infinite recursion
	cout << ", " << _z;
}
```

## protected

So far we've seen two access specifiers: private and public.  There is actually a third called *protected*.  Protected means that members within that section cannot be accessed from externally, but they can be accessed from subclasses.  Let's see how we could have implemented the previous example using protected.

```c++
class Vehicle
{
protected:  // new access modifier
	int _x;
	int _y;
public:
	Vehicle(int const x, int const y);
	void print() const;
	void printPosition() const;
};

void Vehicle::printPosition() const
{
	cout << _x << ", " << _y;
}

void Helicopter::printPosition() const
{
	cout << _x << ", " << _y << ", " << _z;
}
```

Since _x and _y are listed as protected they can be accessed in the Helicopter subclass methods such as printPosition.

Although protected easily fixes this problem, it is probably wise to be a bit cautious in its use.  Members variables that are protected are much more accessible than private members.  They can be accessed or modified by anybody that creates a subclass.  This means that any modification to protected members could potentially result in modification to every subclass.  If the class is shared with other people then modifications become extremely difficult.

There is less concern about creating protected methods then members.

Protected members should be thought of as semi-public extension points.  They are not public in the sense that they can be accessed outside the class definition, but they are public in the sense that they can be arbitrarily modified by any subclass.

## Calling Member Functions

Now that we have our print methods working we can call them as follows:

```c++
Helicopter policeHelicopter(1, 2, 3);
Vehicle crimalCar(1, 2);

policeHelicopter.print();
crimalCar.print();
```

The output is exactly what we'd except:

```
Helicopter at (1, 2, 3)
Vehicle at (1, 2)
```

Remember that when accessed via a pointer or reference we can treat an instance of a derived class as though it were an instance of its base class.  Given that knowledge and our Vehicle and Helicopter classes let's write this another way:

```c++
void printVehicles(Vehicle const * vehicles[], int const count)
{
	for (auto i = 0; i < count; ++i)
	{
		Vehicle const *current = vehicles[i];
		current->print();
	}
}

int main()
{
	Helicopter policeHelicopter(1, 2, 3);
	Vehicle crimalCar(1, 2);

	Vehicle const* vehicles[] = {
		&policeHelicopter,
		&crimalCar
	};

	printVehicles(vehicles, 2);

	return 0;
}
```

This all should make sense.  We're just making an array of vehicles and passing them to the printVehicles function.  This program compiles and runs without error, but it probably doesn't behave as we might desire.

Remember we're telling both policeHelicopter and crimalCar to print.  These objects have different classes that have their own definition of print.

Here's the output:

```
Vehicle at (1, 2)
Vehicle at (1, 2)
```

We might have expected or desired that the the output here would have matched the previous output (where Helicopter and z component were printed for policeHelicopter).

## Binding

To understand why this behaves as it does we must start to understand the concept of binding.

In programming languages *binding* or *name binding* refers to how data or code is associated with an identifier.  Different programming languages have different binding mechanisms and understanding the rules for a language is an important part of learning that language.

As you've learned C++ you've seen numerous examples of binding without really realizing it.  Take a super simple example:

```c++
void foo() { // here's a function named foo
}

foo(); // this call is bound to the function above
```

Binding also happens with classes.  We just saw this:

```c++
Helicopter policeHelicopter(1, 2, 3);
Vehicle crimalCar(1, 2);

policeHelicopter.print();  // binds to Helicopter::print
crimalCar.print();         // binds to Vehicle::print
```

Here the binding happens based on the type of the object.  Since policeHelicopter is a Helicopter it binds to Helicopter::print whereas crimalCar is a Vehicle so it binds to Vehicle::print.

Let's look again at printVehicles:

```c++
void printVehicles(Vehicle const * vehicles[], int const count)
{
	for (auto i = 0; i < count; ++i)
	{
		Vehicle const *current = vehicles[i];
		current->print(); // binds to Vehicle::print
	}
}
```

The call to the print method is bound to Vehicle::print because current's type at compile time is Vehicle.  The problem is that Vehicle could be pointing to a subclass that had its own print.  Regardless we end up calling Vehicle::print instead of the more appropriate Helicopter::print.

The reason this behaves this way is because C++ binding rules normally require that function names be bound to functions at compile time.  This is known as *static binding*.  

In the example above the call to print is bound to a specific print definition, that is Vehicle::print, because the compile time type of current is Vehicle.  current could be pointing to a subclass of Vehicle (Helicopter for example), but at compilation time when the compiler emits code to actually call the print method there is not enough information to do anything other than call Vehicle::print.

## Manual Dynamic Dispatch

To fix the previous problem we need to introduce a concept something called *dynamic dispatch*.  With dynamic dispatch the appropriate print method will be picked based on the run-time type instead of the compile time type.  The code that makes the call must be generated at compile type.  Some of the information needed to select which subclass's implementation should be used cannot be known until runtime.  To work around this problem we need to give the program a little more information and give the call site a little more code to use that information to select the correct implementation.

I'm going to first write out code to make dynamic dispatch happen manually.  This is to give a good conceptual basis for how this works.  Later we'll learn how the C++ compiler makes this a lot easier by doing this work for us.

Before I get started there might be one new concept: function pointers.

<!-- TODO: add something on function pointers if it is new -->

```c++
#include <iostream>
using namespace std;

struct VehicleVTable;
extern VehicleVTable const g_vehicleTable;
extern VehicleVTable const g_helicopterTable;

class Vehicle
{
private:
	int _x;
	int _y;
protected:
	Vehicle(int const x, int const y, VehicleVTable const *const vtable) :
		_x(x),
		_y(y),
		_vtable(vtable)
	{
	}

public:
	VehicleVTable const *const _vtable; // "virtual" table

	Vehicle(int const x, int const y) : Vehicle(x, y, &g_vehicleTable) {
	}

	void print() const {
		cout << "Vehicle at (";
		printPosition();
		cout << ")" << endl;
	}

	void printPosition() const {
		cout << _x << ", " << _y;
	}
};

class Helicopter : public Vehicle
{
private:
	int _z;
public:
	Helicopter(int const x, int const y, int const z) :
		Vehicle(x, y, &g_helicopterTable),
		_z(z)
	{
	}

	void print() const {
		cout << "Helicopter at (";
		printPosition();
		cout << ")" << endl;
	}

	void printPosition() const {
		Vehicle::printPosition();
		cout << ", " << _z;
	}
};

struct VehicleVTable
{
	void (*printVehicle)(Vehicle const *const);
};

void printVehicle(Vehicle const *const vehicle) {
	vehicle->print();
}

VehicleVTable const g_vehicleTable {
	&printVehicle
};

void printHelicopter(Vehicle const *const helicopter) {
	static_cast<Helicopter const *>(helicopter)->print();
}

VehicleVTable const g_helicopterTable{
	&printHelicopter
};

void printVehicles(Vehicle const * vehicles[], int const count)
{
	for (auto i = 0; i < count; ++i)
	{
		Vehicle const *current = vehicles[i];
		current->_vtable->printVehicle(current); // "virtual" call
	}
}

int main()
{
	Helicopter policeHelicopter(1, 2, 3);
	Vehicle crimalCar(1, 2);

	Vehicle const* vehicles[] = {
		&policeHelicopter,
		&crimalCar
	};

	printVehicles(vehicles, 2);

	return 0;
}
```

Let's draw out what we just did.

![](manual-dynamic-dispatch.png)

For each type we define a table that lists all the virtual methods.  We have two instances of our table, one for Helicopter and one for Vehicle.  Every instance of Vehicle has a pointer to a table.  So there is one virtual table for each class, but every instance has a pointer to the vtable.

Calling the virtual method requires accessing the virtual table for the instance and calling the method the table points to.  Notice that now the compile will be able to emit code at compile time that will use extra information from the vtable that is only available at runtime.

This extra work gives us the ability to treat vehicles generically (such as in printVehicles) but still have the appropriate behavior depending on the runtime type of the object being referred to.  This means we can write code that is closed to modification but open for extension.  In the case of printVehicles we could reuse that same function for a new type of Vehicle subclass (say Airplane) without modifying printVehicles.

If this seems like a lot of extra work just so that we can work with types more generically perhaps you're write.  Doing this work manually is a lot of work.  Thankfully the C++ compiler has built in support for dynamic dispath.  The right tooling can make a job that once seemed impractical easy.

## virtual Methods

The compiler provides a mechanism to provide dynamic dispatch automatically using something known as *virtual* methods.  It actually works somewhat like what we did in the previous example, but the compiler does a lot of the work for us.  The compiler creates the virtual table (sometimes known as a vtable) automatically.  It automatically adds a pointer to the class that points to its vtable.

Let's re-write the previous example using virtual methods:

```c++
#include <iostream>
using namespace std;

class Vehicle
{
private:
	int _x;
	int _y;
public:
	Vehicle(int const x, int const y) :
		_x(x),
		_y(y)
	{
	}

	virtual void print() const {
		cout << "Vehicle at (";
		printPosition();
		cout << ")" << endl;
	}

	void printPosition() const {
		cout << _x << ", " << _y;
	}
};

class Helicopter : public Vehicle
{
private:
	int _z;
public:
	Helicopter(int const x, int const y, int const z) :
		Vehicle(x, y),
		_z(z)
	{
	}

	void print() const {
		cout << "Helicopter at (";
		printPosition();
		cout << ")" << endl;
	}

	void printPosition() const {
		Vehicle::printPosition();
		cout << ", " << _z;
	}
};

void printVehicles(Vehicle const * vehicles[], int const count)
{
	for (auto i = 0; i < count; ++i)
	{
		Vehicle const *current = vehicles[i];
		current->print(); // virtual call
	}
}

int main()
{
	Helicopter policeHelicopter(1, 2, 3);
	Vehicle crimalCar(1, 2);

	Vehicle const* vehicles[] = {
		&policeHelicopter,
		&crimalCar
	};

	printVehicles(vehicles, 2);

	return 0;
}
```

A virtual method can be overridden by a base class.  The overridden method must have the same name and parameters as the base class method.  The return type must be either the same or a pointer/reference to a derived type.

Types that have at least one virtual method (this could be on a base type) are called *polymorphic types*.  The dynamic dispatch mechanism is also known as *polymorphism*.  Polymorphism simply means that something occurs in more than one form.  In our example the print method exists in more than one form.  It has a form used by Helicopter and a different form used by Vehicle.

There are different kinds of polymorphism.  C++ supports run-time polymorhism (dynamic dispatch) as well as compile-time polymorphism (templates).  We'll cover templates later.

## Virtual Declaration

In the previous example I put the defintion for the virtual print method in the Vehicle definition to save space.  Normally we would put the class definition in a header and only place method declarations in that class definition.  For example:

```c++
// Vehicle.hpp
#pragma once

class Vehicle
{
private:
	int _x;
	int _y;
public:
	Vehicle(int const x, int const y);

	virtual void print() const;
	void printPosition() const;
};
```

```c++
// Vehicle.cpp
#include "Vehicle.hpp"

Vehicle::Vehicle(int const x, int const y) :
	_x(x),
	_y(y)
{
}

void Vehicle::print() const { // notice no virtual here
	cout << "Vehicle at (";
	printPosition();
	cout << ")" << endl;
}

void Vehicle::printPosition() const {
	cout << _x << ", " << _y;
}
```

The virtual specifier is not part of the type of a function and cannot be repeated in the out of class definition.

## Efficency

When we manually wrote dynamic dispatch we noticed that there was some extra overhead.  Each class instance needed a little extra space for a pointer to its vtable.  There was also a fixed overhead for the vtable itself.  The call sites also involved a little extra overhead due to the extra indirection.

When using virtual methods most of the same overheads still exist.  We need an extra pointer in each class instance (this can be observed by using the sizeof operator).  There is some extra overhead at the call site.

Some programmers get really really worried about the overhead of virtual methods and attempt to avoid them wherever possible.  These fears are generally unfounded.  Compilers are able to emit code for virtual calls that is no more than 25% slower than a normal function call (remember that a normal function call is incredibly fast).  Anywhere we are not concerned about the overhead of a normal function call we are unlikely to suffer performance problems due to virtual calls.

Also remember that we only pay the performance costs of virtual when we choose to use it.  This is important to the design philosophy of C++.  We should only have to pay the performance cost for features that we choose to use.  This is part of why C++ has such good low-level performance.  This doesn't mean we always should feel required to use the most efficient low-level possible.  Most parts of a program are not performance critical.  We should use higher level mechanisms to make those parts easier, and only fallback to lower level mechanisms for critical hot spots.  

Method calls only have virtual overhead when they are marked virtual.  An extra pointer for the vtable is only added when there is a virtual method in the class (or a subclass).  If there are no virtual methods then the vtable itself isn't generated either.

## Explicit Qualification

When we explicitly specify the method using the scope resolution operator, such as printPosition previously, then dynamic dispatch is not used.

```c++
Vehicle::printPosition(); // dynamic dispatch disabled
```

This prevents infinite recursion.


## Virtual Destructors

We can declare our destructors to be virtual.  Here is an example:

```c++
class Container {
public:
	virtual ~Container();
};
```

When we define a type with virtual methods that means we might intend subclasses to be manipulated via the base class interface.  We already saw an example of this in printVehicle.

One way we could manipulate a subclass via its base interface is by deleting it.  For example:

```c++
void foo(Container *container) {
	delete container;
}
```

If Container's destructor is non-virtual then when foo deletes the container only Container's destructor will run.  If one of Container's subclasses had a destructor, this would be possibly disastrous because that subclass's destructor would not run.

Let's see an example.  First let's not use virtual destructors:

```c++
class Container {
public:
	~Container() {
		cout << "~Container()" << endl;
	}
};

class Array : public Container {
private:
	int const *_elements;
	int const _size;
public:
	explicit Array(int size) : _elements(new int[size]), _size(size) {
		cout << "Array(" << size << ") -- buffer allocated" << endl;
	}

	~Array() {
		cout << "~Array() -- buffer deallocated" << endl;
		delete[] _elements;
	}
};

int main()
{
	Container *buffer = new Array(100);
	delete buffer;

	return 0;
}
```

Running the above code we'll see the following output:

```
Array(100) -- buffer allocated
~Container()
```

We allocate the buffer but we never deallocate it.  This is a memory leak!  This happens because we didn't make the destructor virtual.  The default binding rules mean that delete will call the destructor based on the compile-time type of what is deleted (which in the previous case was Container*).

If we change the previous example to use virtual destructors the output will be as follows:

```c++
Array(100) -- buffer allocated
~Array() -- buffer deallocated
~Container()
```

So classes can and most often should have virtual destructors.

When wouldn't a class need a virtual destructor?  Here's some cases:

 - It has no subclasses
 - It has no virtual methods
 - It has no subclasses that have destructors
 - It is never deleted through the base interface
 - It is never heap allocated

Keeping track of and applying all these rules is difficult at best and basically impossible at worst.  A better general rule is to make the destructor virtual for any type that has any virtual method.  This has no extra cost and will prevent member leaks and other bugs.

Some people go even further and say that all destructors should be virtual.  This rule has some additional cost (see section on efficiency) when it is applied to classes that don't already have a virtual method, but the cost is pretty small.

I would recommend to just go ahead and **make all your destructors virtual unless you have a very good reason not to**, for example you need a very specific memory layout or you have a real performance issue.

## Pure Virtual

Marking a function virtual means that a subclass *may* override it, but it doesn't *need* to.  If a subclass doesn't override the base class method then the base class implementation is used.

This pattern is useful for objects that make sense to be used on their own, but that we may wish to extend in a subclass.  We saw an example of this in our Vehicle class.  It is useful on its own, or we can subclass it (as with Helicopter).

Alternatively we can specify that derived classes *must* override a method.  In this case the base class has no implementation.  The implementation must be provided by a subclass.  This is called a *pure virtual* method.

If a type has any pure virtual methods then the type is called *abstract*.  Abstract types are not useful themselves.  They serve only as an interface that derived classes must follow.

As an example let's consider a *stream*.  Streams are highly useful in computer science.  We can think of a stream as an arbitrary sequence of bytes that we can read or write to.  You've already seen many examples of streams.  cout is a stream that writes to standard output (the console).  cin is a stream that reads from standard input (the standard input).  You've seen fstream which is a stream that can read/write to a file.  Streams can be designed to read/write data for interprocess communication, networking, serial ports.

For simplicity let's define our own set of extremely simple stream classes:

```c++
class Stream { // abstract type
public:
	virtual int read(char *buffer, int size) = 0; // pure virtual
	virtual void write(char const *buffer, int size) = 0; // pure virtual

	virtual ~Stream(); // virtual destructor
};

class FileStream : public Stream {
public:
	int read(char *buffer, int size);
	void write(char const *buffer, int size);
};

class MemoryStream : public Stream {
public:
	int read(char *buffer, int size);
	void write(char const *buffer, int size);
};
```

Notice the syntax--the read/write methods on Stream are declared with:

```c++
= 0
```

at the end.  These methods may have no body.  They have no definition.

Stream is not useful in a concrete sense--it is abstract.  It describes the general idea of a stream.  If Stream was not abstract then we would need to provide a definition of the read/write methods.  But what possible definition would make sense?  Streams need to actually read/write to something.  There is no reasonable definition that could be provided for the abstract concept of a stream.

Let's write a function that uses a Stream:

```c++
void writeTrace(TraceLevel level, char const *const message, Stream &stream)
{
	auto const code = toCode(level);
	stream.write(&code, 1);
	stream.write(": ", 2);

	stream.write(message, strlen(message));
}
```

This function writes a trace message out to stream.  This function is responsible for formatting and sending the appropriate bytes to the stream that is passed in.

But what does writeTrace actually write to?  Is it a file, the console, an array in memory, a serial port, a network connection, an HTTP response (for a web site), or something else?  The answer could be any of them, or anything else.  writeTrace could be used to write data to any of those things, or even something else, without needing to change writeTrace at all.

Hopefully this gives you a feel for one of the most important aspects of OOP--we can design software that can support new features through extension instead of modification.  If we wrote this without OOP we might have to re-write the writeTrace function for file and console.  Then later we might need to write to a new type of stream and have to modify it again.

When a class is abstract (it has at least 1 pure virtual method) then we cannot instantiate it.  Attempting to do so will result in a compiler error:

```c++
Stream s; // ERROR: cannot instantiate abstract class 
```

## Inheritance Styles

Now that we've seen a few examples of derived types we're at the point were we can think more generally about derived types.  The language features in C++ support two styles of inheritance:

### Implementation Inheritance

Here we're simply avoiding duplication between different classes by sharing some of the implementation within a base class.  The initial examples using Vehicle and Helicopter were examples of this style.

### Interface Inheritance

Here we use the base class simply to define an interface that subclasses will support.  This allows us to write parts of our programs against generic abstractions.  By defining an base class as an interface we can treat all subclasses as interchangeable units.

## Interface

We can combine interface inheritance and implementation inheritance in the same class (i.e. a class with implementation and pure virtual methods), however this should be done with care.  It is usually simpler to avoid this mixing.

Classes that contain only pure virtual methods and no implementation are frequently referred to as *interfaces*.  There is, perhaps unfortunately, no way to indicate directly that a type is an interface in C++ (unlike langauges such as Java or C#) so an interface isn't so much a language feature as it is a common pattern.

We've already seen an interface:

```c++
class Stream {
public:
	virtual int read(char *buffer, int size) = 0;
	virtual void write(char const *buffer, int size) = 0;

	virtual ~Stream() {}
};
```

Note that the presence of an empty non-pure virtual destructor is typically allowed in an interface.

A more complete interface declaration would also disable copy/move operations.  Since an interface is meant to be manipulated via a pointer or reference these operations would be problematic.

```c++
class Stream {
public:
	virtual int read(char *buffer, int size) = 0;
	virtual void write(char const *buffer, int size) = 0;

	Stream() = default;
	Stream(Stream const &) = delete;
	Stream & operator=(Stream const &) = delete;
	Stream(Stream &&) = delete;
	Stream & operator=(Stream &&) = delete;

	virtual ~Stream(); // virtual destructor
};
```

Interfaces are also sometimes called *protocols* (e.g. Objective-C).

It is relatively common to see a convention of prefixing interface names with an I to indicate that the type is an interface.  For example:

```c++
class IStream;
```

## override

Overriding can be confusing and error prone.  If we define a method with the same signature in both the base class and a derived class then:

- If the method is virtual in the base class is virtual then the method in the derived class overrides the method.
- If the method is not virtual in the base class than the base class method is hidden by the derived class.

This all works fine and well but what if we accidentally have something off in the signature?  Or what if we change something in the signature of the base class and forget to update the derived classes?  Normally this isn't too bad to deal with for small hierarchies, but it can get confusing for larger class hierarchies.

Let's see an example:

```c++
class A {
public:
	virtual void f();
};

class B : public A {
	void f(); // B::f overrides A::f (and is virtual)
};
```

We could also write the following:

```c++
class A {
public:
	virtual void f();
};

class B : public A {
	virtual void f(); // more explicit about being virtual
};
```


B::f is implicitly virtual because it overrides the base class's virtual method f.  Now let's assume that later we make some change to A:


```c++
class A {
public:
	virtual void f(int a); // added a parameter
};

class B : public A {
	void f(); // B::f no longer overrides A::f
};
```

Wouldn't it be nice if the compiler could report an error in that situation?  The problem is that the compiler lacks the required information to know what the programmer's intent was.  As far as the compiler is concerned the programmer didn't intend for B::f to override anything.

Luckily in C++11 a language feature is available that allows the programmer to specify his intent.  When the intent is clear than the compiler can provide useful error messages:

```c++
class A {
public:
	virtual void f();
};

class B : public A {
	void f() override; // clarify our intent, B::f should override something
};
```

Now let's make a change to A:

```c++
class A {
public:
	virtual void f(int a);
};

class B : public A {
	void f() override; // ERROR: B::f doesn't override anything
};
```

override goes at the end vs virtual which goes at the end of the function.  This isn't logical, but it is done for compatibility reasons.

## final

Occasionally we might have the need to start out with a base class having virtual methods, but then have a subclass prohibit additional overriding.

For this purpose we can use the keyword final.  final prevents further overriding.  We can also make all methods of a class final by marking the class as final.  Marking the class final additionally prevents any subclassing.

Here's an example:

```c++
class Shape {
public:
	virtual void draw() const;
};

class Line : public Shape {
public:
	void draw() const final override; // draw cannot be overriden further
};

class Triangle final : public Shape {
public:
	void draw() const override;
};

class Bar : public Triangle { // ERROR: cannot derive from Triangle
public:
	void draw() const override;
};

class Foo : public Line {
public:
	void draw() const override; // ERROR: cannot override
};
```

It is somewhat unusual to use final.  One example of its usage would be a program where you had a set of classes to represent elements of a programming language.  The elements might derive from a common base class.  Deriving new types from leaf elements should be prohibited because it would allow users to modify the language meaning.

The basic idea is that virtual methods should be thought of as extensibility points for subclasses.  final provides a mechanism to disallow extensibility.

## Contextual Keyword

One interesting language aspect of override and final is that they are *contextual keywords*.  That is they are keywords only when found in the places those keywords would be expected.  Contextual keywords are common when languages are extended to prevent new keywords from breaking old programs.

Prior to C++11 final/override were not part of the language so those identifiers could have been used for a variable.  By making these keywords contextual, old programs can still be compiled even if they happened to use final or override as an indentifier.  To avoid confusion it would be recommended to avoid using these identifiers in new programs.

## using Base Members

Method overloads do not overload across classes.  This can lead to confusing behavior/errors.  Consider the following:

```c++
class Base {
public:
	void foo(int a) {
		cout << "Base::foo(" << a << ");" << endl;
	}
};

class Derived : public Base {
public:
	void foo(char const *const str) {
		cout << "Base::foo(\"" << str << "\");" << endl;
	}
};


int main()
{
	Derived d;
	d.foo(1);  // ERROR: no foo overload taking an int

	return 0;
}
```

Depending on the parameter types involved this could also result in an unexpected overload getting called (instead of a compile-time error).

We can pull Base::foo into Derived with a using statement:

```c++
class Base {
public:
    void foo(int a) {
        cout << "Base::foo(" << a << ");" << endl;
    }
};

class Derived : public Base {
public:
    using Base::foo;
    void foo(char const *const str) {
        cout << "Base::foo(\"" << str << "\");" << endl;
    }
};


int main()
{
    Derived d;
    d.foo(1);  // WORKS: calls Base::foo(int)

    return 0;
}
```

## Base Class Access Specifier

In all the previous examples we've always specified inheritance via:

```c++
: public Base
```

Like member variables, we can use any access specifier there.  So any of the following would also be valid:

```c++
class A : public Base {};
class B : protected Base {};
class C : private Base {};
```

Like member variables the default access specifier for classes is default and the default for structs is public:

```c++
class  D : Base {}; // Base is a private base class
struct E : Base {}; // Base is a public  base class
```

public is by far the most common access specfier for inheritance.  public is the only access specifier that expresses an is-a relationship.  public inheritance should always be used unless you have a good reason not to.

So what would be a good reason not to?  C++ has a nasty habbit of making things a little to complex for their own good, but there are some possible cases to use non-public inheritance.

First of all what in the world does non-public inheritance even mean?  First let's consider an example that doesn't even use non-public inheritance.

```c++
class Array {
private:
    int * _buffer;
	int   _size;
public:
    int elementAt(unsigned int const index) const;
    void setElement(unsigned int const index, int value);
    unsigned int size() const;
};

class Set1 {
private:
	Array _array;
public:
	bool add(int const value);  // calls _array.setElement
	int  get(int const index) const; // calls _array.elementAt
	unsigned int size() const; // calls _array.size
};
```

We have two incomplete classes an Array and Set1.  Array represents an array whose size is dynamic (but unchangeable).  Set1 represents a mathematical set.  It can contain only unique elements.

Notice that we implement the Set1 using an Array.  We can say that a Set1 has an Array.  We implement Set1 by calling methods in Array.  That is we implement Set1 in terms of Array.  This is called *containment*--Set1 contains an instance of Array.

Alternatively we could do something as follows:

```c++
class Set2 : private Array {
public:
	bool add(int const value); // calls setElement
	int  get(int const index) const; // calls elementAt
	using Array::size;
};
```

Set1 and Set2 have no substantial difference.  So both containment and non-public inheritance allow a type to be implemented using another type.  They are both used to express an is-implemented-in-terms-of relationship.

Non-public inheritance implements a class by depending on the public and protected parts of the base class.  Containment implements a class by depending on only the public parts of the contained class.

Non-public inheritance is a stronger form of coupling and should therefore be avoided unless required.  Why might it be needed?

- Protected member access is needed
- We need to override a virtual method (e.g. the is-implemented-in-terms-of class is abstract)

In addition to the previous, we could use non-public inheritance to express a limited form of is-a which only applied within the members of that class.  For example Set2 is-a Array inside its own member variables, but is not an Array in terms of public.

protected inhertance works like private, but it allows the base class to be exposed to further subclasses where private does not.

Non-public inheritance is a strange feature of C++ that is not common in OOP languages.  It is quite rare to see it used.  Its complexity and likelihood for confusion probably do not justify its inclusion in the language.