

CMake is a platform independent build system.

It is used to compile executables and build installers.


There is loads and loads to this. Here is a simple example.

In a directory called LearnCMake there are the following 3 files:

HelloWorld.cpp:
```
#include <iostream>
#include "HelloWorld.h"

// Constructor definition
HelloWorld::HelloWorld(const std::string& message) : message(message) {}

// Method definition
void HelloWorld::printMessage() const {
    std::cout << message << std::endl;
}

// Main function
int main() {
    HelloWorld hello("Hello, World!");
    hello.printMessage();

    // Await key press before exiting
    std::cout << "Press Enter to exit...";
    std::cin.get();

    return 0;
}
```

HelloWorld.h:
```
#ifndef HELLOWORLD_H
#define HELLOWORLD_H

#include <string>

class HelloWorld {
public:
    // Constructor
    HelloWorld(const std::string& message);
    
    // Method to print the message
    void printMessage() const;

private:
    std::string message;
};

#endif // HELLOWORLD_H
```

CMakeLists.txt:
```
cmake_minimum_required(VERSION 3.10)

# set the project name
project(LearnCMake)

# Add an executable
add_executable(HelloWorld HelloWorld.cpp)
```

1. Open a command prompt scoped to LearnCMake directory

2. Type the command to create the build files:
```
cmake -S . -B build -A x64 -DCMAKE_BUILD_TYPE=Debug
```
3. Type the command to build:
```
cmake --build build --config Debug
```

Similarly for a release build:
```
cmake -S . -B build -A x64 -DCMAKE_BUILD_TYPE=Release
cmake --build build --config Release
```

Explanation of the command line parameters:
- `-S .` specifies the source directory (current directory).
- `-B build` specifies the build directory.
- `-A x64` or `-A Win32` specifies the architecture (x64 for 64-bit and Win32 for 32-bit).
- `-DCMAKE_BUILD_TYPE=Debug` or `-DCMAKE_BUILD_TYPE=Release` specifies the build type.