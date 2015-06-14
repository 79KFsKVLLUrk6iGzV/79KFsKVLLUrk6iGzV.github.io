# Self-Reference (this)

In non-static member functions we can access the instance from which the method was invoked via the **this** pointer.  The this pointer is read-only.  Remember that class members implicitly have a reference to their own instance.  The this pointer is an explicit reference.  *Normally you do not need to use the this pointer*, but it is occasionally useful.

## Name Disambiguation

One use of the this pointer is to disambiguate names.  Consider the following:

```c++
class Date {
private:
	int year;
public:
	void setYear(int year);
};

void Date::setYear(int year) {
	this->year = year; // assign the parameter to the member variable
}
```

Date's constructor has a parameter named year, but there is also a member variable named year.  Using the this pointer allows us to specify which one we meant.

Instead of using this to disambiguate, I typically use a underscore prefix convention to indicate member variables.  Consider:

```c++
class Date {
private:
	int _year;
public:
	void setYear(int year);
};

void Date::setYear(int year) {
	_year = year;
}
```

The this pointer can also be used to call methods.  Again, normally this isn't needed or required as it is done implicitly.

## Method Chaining

The this pointer can also be used to allow method chaining.  Consider the following:

```c++
class Date {
private:
	int _year;
	int _month;
public:
	Date& setYear(int year);
	Date& setMonth(int month);
};

Date& Date::setYear(int year) {
	_year = year;
	return *this;
}

Date& Date::setMonth(int month) {
	_month = month;
	return *this;
}

int main() {
	Date date;
	date.setYear(1992).setMonth(4); // method chaining

	return 0;
}
```

## Passing this

The this pointer can also be used to pass oneself to an something outside your class.  For example you could call a non-member function that needed a reference a class instance.  Or you might want to pass yourself to another class.  We'll see an example of that in the next section.