# **Arithmetic Operators**

**Addition:**

```
int sum = a +b;
```

**Subtraction:**

```
int diff = a-b;
```

**Multiplication:**

```
int prod = a*b;
```

**Division:**
```
int div = a/b;
```

**Modulus:**
Calculates the remainder of an integer division.
```
int remainder = a % b;
```

**Increment:**
Can be used prefix (increments the value before returning it) or postfix (returns the value first and then increments it)
```
int x = 5;
int y = ++x; // x = 6, y = 6
int z = x++; // x = 7, z = 6
```

**Decrement:**
Can be used prefix or postfix, similar to increment.
```
int x = 5;
int y = --x; // x = 4, y = 4
int z = x--; // x = 3, z = 4
```

**cmath** 
There are more advanced arithmetic functions included in the cmath library, such as exponents.

```
#include <iostream>
#include <cmath>

int main() {
    double base = 2.0;
    double exponent = 3.0;
    double result = pow(base, exponent);
    
    std::cout << base << " raised to the power of " << exponent << " is " << result << std::endl;

    return 0;
}
```

# **Logical Operators**

**AND:**
Returns true if both conditions are true.

syntax:
```
(expression1 && expression2)
```

example:
```
int a = 5, b = 10;
if (a > 0 && b > 0) {
    std::cout << "Both values are positive." << std::endl;
}
```

**OR:*
Returns true if either one of or both of the expressions are true.

syntax:
```
(expression1 || expression2)
```

example:
```
int a = 5, b = -10;
if (a > 0 || b > 0) {
    std::cout << "At least one value is positive." << std::endl;
}
```

**NOT:**
Reverses the result of the condition/expression applied to.

syntax:
```
!(expression)
```

example:
```
int a = 5;
if (!(a < 0)) {
    std::cout << "The value is not negative." << std::endl;
}
```


# **Bitwise Operators**

Bitwise operations directly manipulate the bits of a number. Can be super useful for optimising algorithms and manipulating memory.

**Bitwise AND (&)**
Takes 2 numbers and compares them bit by bit.

example:
```
int result = 5 & 3;
```

The result will be 1:
0000 0101 = 5
0000 0011 = 3
            &
0000 0001 = 1 


**Bitwise OR (|)**
Takes 2 numbers and compares them bit by bit. 

example:
```
int result = 5 | 3;
```

The result will be 7:
0000 0101 = 5
0000 0011 = 3
			|
0000 0111 = 7 

**Bitwise XOR(^)**
Takes 2 numbers and compares them bit by bit.  Returns a 1 only when there is a single 1.

example:
```
int result = 5 ^ 3;
```

The result will be 6:
0000 0101 = 5
0000 0011 = 3
			^
0000 0110 = 6

**Bitwise Not (~)**
Takes 1 number and inverts the 1s and 0s.

example:
```
int result = ~5;
```

result will be -6 (2s complement):
0000 0101 = 5
			~
1111 1010 = -6

**Bitwise Left Shift (<<)**
Shifts the bits in the left direction by a specified amount. Vacated bits are populated with zeros.

example :
```
int result = 5 << 1;
```

result will be 10:
0000 0101 = 5
			<< 1
0000 1010 = 10


**Bitwise Right Shift (>>)**
Shifts the bits in the right direction by a specified amount. The vacated bits are filled with zeros or sign bit depending on the input value being signed or unsigned.

example:
```
int result = 5 >> 1;
```

result will be 2:
0000 0101 = 5
			>> 1
0000 0010 = 2

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
