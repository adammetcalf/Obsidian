
There exist 2 types of memory used by a cpp program.

Each byte of a cpp program has a memory address, ie 0x00003331. 0x00000000 has is a low number, and 0xFFFFFFFF is a high number. Memory is laid out as such:![[CppMemory.png]]

# **Stack**

The Stack is at the top of the memory with high address. Each time a function is called, the machine allocates some stack memory. The memory is automatically deallocated when the function is complete. This memory may be reused

*A common mistake is to return a pointer to a stack variable in a helper function.* This is is a mistake because the data held in this memory location may change, and the pointer is essentially pointing to garbage.

Either a variable must be returned by copy (ie not a pointer) or the value must be placed in memory that is less dynamic (ie the Heap).

# **Heap**

Heap memory is assigned explicitly by programmers, and will not be deallocated until explicitly freed.

Heap memory is allocated using the keyword `new`. 

Note that if Heap memory is not deallocated using the keyword `delete`, there will be a memory leak. These are bad and can cause a program to crash.

*If you try to use pointers to heap memory that has been released, you get undefined behaviour*. This can be combatted by setting the value of freed pointers to `nullptr` immediately after `delete`.


ie:

```
Cube *CreateCubeOnHeap(){
	Cube *c = new Cube(20);        
	return c;                      
}

int main(){
	Cube *cube = CreateCubeOnHeap();  
	double v = cube->getVolume();
	delete cube;
	cube = nullptr;
	return 0;
}

```


# **Conclusion**

The stack is dynamic memory, the programmer has no control of this.

The heap is memory that the programmer can actively use, though care must be taken to avoid memory leaks.
