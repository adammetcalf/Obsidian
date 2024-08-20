
# **Fundamental Types**

There are only 4 fundamental datatypes. Everything else is derived from these.

Integers (the size of ints depend on the system architecture, usually 4 bytes)
```
int num = 6;
```

Float provides single precision floating point numbers (again, usually 4 bytes)
```
float pi = 3.14f;
```

Double provides double precision numbers (usually 8 bytes)
```
double pi = 3.1415926535
```

Characters represent a single character, and are stored using the ASCII value of the symbol. Typically use 1 byte of memory.
```
char letter = 'A';
```

Booleans are logical values, and occupy 1 byte of memory:
```
bool isAdamHot = true;
```


# **Derived Data Types** 

Arrays are used to store multiple values of the same datatype in consecutive memory locations.
```
int numbers[5] = {1,2,3,4,5}
```

In c++, a standard array is not dynamic. I.e. it must have a length when declared, and the size cannot be changed at runtime.

[Pointers and references](obsidian://open?vault=Obsidian&file=CPP%2FLessons%2FPointers) are also considered derived datatypes.

**User Defined Data Types**

Structs are used to store different datatypes under a single variable. You may consider them as a class without any functionality. A struct does not store all variables in the same memory location.
```
struct Person {
    std::string name;
    int age;
    float height;
};

Person p1 = {"John Doe", 30, 5.9};
```

A union is like a struct but all members share the same memory location.
```
union Data {
    int num;
    char letter;
    float decimal;
};

Data myData;
myData.num = 42;
```

Struct Vs Union Usage: 

- **`struct`:** Use a `struct` when you need to group different types of variables together and access all of them simultaneously.
- **`union`:** Use a `union` when you need to store different types of data in the same memory location but only one type of data will be used at any given time.

A class contains data, and also functions to interact with that data. This is getting into OOP territory, which I am already familiar with.
```
class Person {
public:
    std::string name;
    int age;

    void printInfo() {
        std::cout << "Name: " << name << ", Age: " << age << std::endl;
    };
};

Person p1;
p1.name = "John Doe";
p1.age = 30;
```


There are further [complex derived datatypes](obsidian://open?vault=Obsidian&file=CPP%2FLessons%2FComplex%20Derived%20Data%20Types) relying on implementations of the standard library.