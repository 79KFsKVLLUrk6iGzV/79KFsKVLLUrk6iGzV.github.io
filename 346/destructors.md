---
title: Destructors
---
# Destructors

We can also define special methods that can be used for cleanup.  Destructors are automatically called when the object is destroyed (when it goes out of scope or is deleted).  Since destructors can be thought of as the complement of constructors, they use the same basic syntax except with the complement operator ~.

```c++
#include <iostream>

using namespace std;

class Array {
private:
    int * buffer;
public:
    explicit Array(int size);
    ~Array();
};

Array::Array(int size) : buffer(new int[size]) {
    cout << "made a buffer of size " << size << endl;
}

Array::~Array() {
    delete [] buffer;
    cout << "buffer cleaned up" << endl;
}

int main(int argc, const char * argv[]) {
    cout << "Enter Count: ";
    
    int count;
    cin >> count;
    
    Array a(count);
    
    cout << "thank you and have a nice day!" << endl;
}
```

```
Enter Count: 10
making buffer of size 10
thank you and have a nice day!
buffer cleaned up
```

In this example we use the constructor to allocate memory from the free store (heap).  The destructor for Array automatically cleans up the memory when it goes out of scope (after the end of main).

Destructors never have arguments.

Destructors are also called automatically when heap allocated variables are deleted.  Consider the following:

```c++
Array * makeArray(int size) {
    return new Array(size);
}

int main(int argc, const char * argv[]) {
    cout << "Enter Count: ";
    
    int count;
    cin >> count;
    
    Array *a = makeArray(count);
    // ...
    delete a;
    
    cout << "thank you and have a nice day!" << endl;
}
```

```
Enter Count: 42
making buffer of size 42
buffer cleaned up
thank you and have a nice day!
```

This constructor/destructor pattern is known as *Resource Acquisition is Initialization* or *RAII* for short.  It can be used to manage more than just memory.  It can manage files, network connections, or any other kind of resource.  This is an extremely powerful mechanism of C++.  Once truly mastered it is one of the best parts of the language.

Note that properly using the types within the C++ standard library eliminates the need for manually calling new/delete.  We'll get to that later.