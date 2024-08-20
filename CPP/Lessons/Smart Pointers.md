Smart pointers are better in almost every way than raw pointers, since there are underlying mechanisms to defend against memory leaks.

There are 3 types of smart pointer:

1. Unique Pointer
2. Shared Pointer
3. Weak Pointer

If using smart pointers, you must include the memory library.
```
#include <memory>
```

# **Unique Pointer** 

Allows exactly 1 owner of the underlying pointer.

```
#include <iostream>
#include <memory>  // Required for std::unique_ptr

// A simple class
class MyClass {
public:
    MyClass() {
        std::cout << "MyClass Constructor" << std::endl;
    }
    ~MyClass() {
        std::cout << "MyClass Destructor" << std::endl;
    }

    void display() const {
        std::cout << "Hello from MyClass!" << std::endl;
    }
};

int main() {
    // Create a unique pointer to an instance of MyClass
    std::unique_ptr<MyClass> ptr = std::make_unique<MyClass>();

    // Use the pointer
    ptr->display();

    // The unique pointer automatically deletes the object when it goes out of scope
    return 0;
}

```

Declaring a unique pointer is as follows:
`std::unique_ptr<TypeOfObjectPointedTo> NameOfPointer = =std::make_unique<TypeOfObject>()`

# **Shared Pointer**
Can have multiple owners of a pointer. Each time a new owner is instantiated, a secret counter increments. Each time an owner is destroyed, the counter is decremented. When the counter reaches zero, the memory is deallocated.

Declaring Shared Pointers:
```
#include <iostream>
#include <memory>  // Required for std::shared_ptr

// A simple class
class MyClass {
public:
    MyClass() {
        std::cout << "MyClass Constructor" << std::endl;
    }
    ~MyClass() {
        std::cout << "MyClass Destructor" << std::endl;
    }

    void display() const {
        std::cout << "Hello from MyClass!" << std::endl;
    }
};

int main() {
    // Create a shared pointer to an instance of MyClass
    std::shared_ptr<MyClass> ptr1 = std::make_shared<MyClass>();

    {
        // Create another shared pointer that shares ownership of the same MyClass instance
        std::shared_ptr<MyClass> ptr2 = ptr1;

        // Use the pointer
        ptr2->display();

        // Use count() to see how many shared_ptr instances own the object
        std::cout << "ptr1 use count: " << ptr1.use_count() << std::endl;
        std::cout << "ptr2 use count: " << ptr2.use_count() << std::endl;
    } // ptr2 goes out of scope here, but ptr1 still holds the object

    // After ptr2 is out of scope
    std::cout << "ptr1 use count after ptr2 is out of scope: " << ptr1.use_count() << std::endl;

    return 0;
}

```

# **Weak Pointer**
Like a shared pointer, but the secret counter is neither incremented or decremented. Use when you want to observe an object, but you don't want to interfere with its lifespan.

Declaring a weak pointer:
```
#include <iostream>
#include <memory>  // Required for std::shared_ptr and std::weak_ptr

// A simple class
class MyClass {
public:
    MyClass() {
        std::cout << "MyClass Constructor" << std::endl;
    }
    ~MyClass() {
        std::cout << "MyClass Destructor" << std::endl;
    }

    void display() const {
        std::cout << "Hello from MyClass!" << std::endl;
    }
};

int main() {
    // Create a shared pointer to an instance of MyClass
    std::shared_ptr<MyClass> sharedPtr = std::make_shared<MyClass>();

    // Create a weak pointer from the shared pointer
    std::weak_ptr<MyClass> weakPtr = sharedPtr;

    // Check if weak pointer can be locked (i.e., if the object still exists)
    if (auto tempSharedPtr = weakPtr.lock()) {
        // If lock() succeeds, weakPtr is promoted to a shared_ptr
        tempSharedPtr->display();
        std::cout << "Object is still alive. Shared pointer count: " << sharedPtr.use_count() << std::endl;
    } else {
        std::cout << "Object has been destroyed." << std::endl;
    }

    // Reset the shared pointer, which will destroy the managed object
    sharedPtr.reset();

    // Try locking the weak pointer again after the shared pointer has been reset
    if (auto tempSharedPtr = weakPtr.lock()) {
        tempSharedPtr->display();
    } else {
        std::cout << "Object has been destroyed after resetting shared pointer." << std::endl;
    }

    return 0;
}

```