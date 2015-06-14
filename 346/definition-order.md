# Definition Order

Classes in C++ allow members to be written in any order.  Within a class we can use a member variable or method of that class before it is defined.  For example:

```c++
class Car {
private:
	int speed;
public:
	void setSpeed(int const s) {
		speed = s; // should be in member initializer list
	}
};
```

Above the member variable speed is defined before it is used, but we could have also written it as follows:

```c++
class Car {
public:
	void setSpeed(int const s) {
		speed = s; // should be in member initializer list
	}
private:
	int speed;
};
```

Some people prefer to list public parts of the class definition first so that readers will see the most relevant parts (the interface) first, but this is somewhat a matter of style.