
# **Stack, Heap and Pointers** 

*A common mistake is to return a pointer to a stack variable in a helper function.* This is is a mistake because the data held in this memory location may change, and the pointer is essentially pointing to garbage.

Either a variable must be returned by copy (ie not a pointer) or the value must be placed in memory that is less dynamic (ie the Heap).



If Heap memory is not deallocated using the keyword `delete`, there will be a memory leak. These are bad and can cause a program to crash.

*If you try to use pointers to heap memory that has been released, you get undefined behaviour*. This can be combatted by setting the value of freed pointers to `nullptr` immediately after `delete`.



Use smart pointers where possible to ensure garbage collection.

# **Bitwise Operations**

**Bitwise Use Cases**
The usecases for bitwise operations are varied. One usecase is to have a bitmask responsible for turning on/off complex bits of hardware in a DAQ system.

One of the clearest use cases is efficient multiplication/division by powers of 2:
- **Multiply by 2^n**: `x << n;`
- **Divide by 2^n**: `x >> n;`

Another usecase is swapping 2 numbers without using a temporary variable:
```
a ^= b; 
b ^= a; 
a ^= b;
```

# Header Guards and pragma once

In the header files, there are 2 methods to guard the files to ensure that functions and variables are protected from being defined more than once.

```
#pragma once
```
Pragma once tells the compiler to only ever include these definitions once. It is a neat way of doing this, but isn't supported in all compilers.

```
#ifndef _NAME_
#def _NAME_

// code here



#endif
```

Header guards do the same thing, but are more verbose. They are however supported by the standard across all compilers.