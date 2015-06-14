# Static Members

We can specify members that are part of the class, but not specific to any instance.

## Static Member Variables

We can define member variables that are part of a class, but shared by all instances of that class by using the **static** keyword.

Take the following example:

```c++
#include <iostream>
using namespace std;

class Student {
private:
	static int _lastID;
	int _id;
public:
	explicit Student();

	int id();
};

Student::Student() {
	_id = ++_lastID;
}

int Student::id() {
	return _id;
}

int Student::_lastId = 0;

int main() {
	Student student1;
	Student student2;

	cout << "student 1: " << student1.id() << endl;
	cout << "student 2: " << student2.id() << endl;

	return 0;
}
```

With non-static (instance) member variables, each instance (e.g. student1, student2) its own member variables (e.g. _id).  With static member variables, the same variable is shared among all instances.  This means that if we make 100 instances of Student, there will be 100 Student::_id, but exactly 1 Student::_lastId.

## Static Member Variable Initialization

Unlike instance variables, static variables listed in the class definition are only declarations--they do not define the variable.  Consider the following:

```c++
// Person.hpp
class Person {
private:
    static int instanceCount; // declaration
public:
    Person();
    ~Person();
};
```

```c++
// Person.cpp
#include "Person.hpp"

Person::Person() {
    ++instanceCount;
}

Person::~Person() {
    --instanceCount;
}
```

```c++
// main.cpp
#include <iostream>
using namespace std;

int main() {
	cout << "about to make a person..." << endl;
    Person p;
    return 0;
}
```

The instanceCount variable within the Person definition is only a declaration.  We've only told the compiler that instanceCount exists, but it has no idea where.  Hopefully there is a defintion somewhere in the program.  If there is then the linker will be able to connect where we use instanceCount to the place where instanceCount is defined.  If there isn't a defintion anywhere then the linker will report an error to us.

In the example above there is no defintion.  On clang we'll get the following error:

```
Undefined symbols for architecture x86_64:
  "Person::instanceCount", referenced from:
      Person::Person() in main.o
      Person::~Person() in main.o
```

To fix this problem we need a definition of instanceCount somewhere in our program.  A good place for it would be in Person.cpp.  Let's modify Person.cpp to include the defintion:

```c++
// Person.cpp
#include "Person.hpp"

int Person::instanceCount;

Person::Person() {
    ++instanceCount;
}

Person::~Person() {
    --instanceCount;
}
```

We could also assign it to something as follows:

```c++
int Person::instanceCount = 0;
```

or even:

```c++
// Person.cpp
#include "Person.hpp"
#include <iostream>

using namespace std;

namespace
{
	int getInstanceCount() {
		cout << "when does this print??" << endl;
		return 0;
	}
}

int Person::instanceCount = getInstanceCount();

Person::Person() {
    ++instanceCount;
}

Person::~Person() {
    --instanceCount;
}
```

```
when does this print??
about to make a person...
```

Crazy huh!?!  The cout statement from inside getInstanceCount executes *before* main!  Turns out main isn't always where your code starts executing.

This also brings up an important thing to be careful about with regard to static member initialization.  We should be extremely careful that code to initialize one static variable is not in any way dependent upon another static member variable.  This is actually true for all static data (which also includes global variables).

It is possible to deal with these problems to some degree (cout itself actually has to deal with this--it's a global variable), but it is better to just avoid if possible.  For more information search for [static initialization order fiasco](https://www.google.com/webhp?q=static%20initialization%20order%20fiasco).

## Static Methods

We can also define static methods.  A static method is not called from any instance of the object, but from the class.  static methods cannot access non-static (instance member variables.

We could define a static method to reset _lastId back to 0.

```c++
#include <iostream>
using namespace std;

class Student {
private:
	static int _lastId;
	int _id;
public:
	explicit Student();

	int id();
	static void reset();
};

Student::Student() {
	_id = ++_lastId;
}

int Student::id() {
	return _id;
}

void Student::reset() {
	//_id = 0;  // illegal reference to non-static member
	//id();     // illegal call of non-static member function

	_lastId = 0;

	Student student;
	student._id = 42; // legal (members can access private members)
}

int Student::_lastId = 0;

int main() {
	Student student1;
	Student student2;

	cout << "student 1: " << student1.id() << endl;
	cout << "student 2: " << student2.id() << endl;

	Student::reset();

	Student student3;
	Student student4;

	cout << "student 3: " << student1.id() << endl;
	cout << "student 4: " << student2.id() << endl;

	return 0;
}
```

Be careful with static variables.  Although their access and usage can be controlled (via a class interface and access modifiers) better than global variables, they are still essentially global variables.