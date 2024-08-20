
# **What is a Pointer? 

Fundamentally, a pointer is just a variable that stores the memory address of a object:
![[PointersMemory.png]]

A pointer has 3 main purposes:
1. To allocate new objects on the heap.
2. To pass functions to other functions.
3. To iterate over elements in arrays or other data structures.

A practical use for a pointer is to pass large data structures into a function. This allows operations to be performed on the data structure, which only has to exist in one place. This prevents a *copy* of the data structure being passed to the function, which has overhead and can be slow.


```
1. int x  = 4; 
2. int *pointX = &x;   
3. int y = *pointX;

```

1. Assigning a variable named 'x' with a value of 4.
2. Assigning a pointer to memory of 'x', called 'pointX'. 

Essentially, 'pointX' is a variable that tells us the memory location of the variable x.

3. Assigning a variable named 'y', with a value of the data held at memory location x (ie, y = 4 in this case). Note that we have take 'pointX' and de-referenced it, which means that the program has looked at the memory location held by variable 'pointX' and returned the value held at that memory location.


Smart pointers were introduced in some form to c++ in 1998. [Smart pointers are documented here.](obsidian://open?vault=Obsidian&file=CPP%2FLessons%2FSmart%20Pointers)

