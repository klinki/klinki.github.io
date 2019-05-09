---
published: true
title: C++ placement new operator and memory layout
---
I have discovered placement `new` operator when I worked on my Java Virtual Machine. It is useful when you need to dynamically create new instance of class and place it into already allocated memory. This is not so common scenario, but it is necessary if you want to write your own memory allocation system.

Here is short code example, how to use it:


```c++
#include <iostream>
using namespace std;

class A
{
  int data = 0;
  A()  { cout << "Constructor called" << endl; }
  ~A() { cout << "Destructor called" << endl; }
}

int main(int argc, const char** arv)
{
	const int MEMORY_SIZE = 65536; // 64 kB
	char* rawMemory = new char[MEMORY_SIZE];
	// allocating own memory pool

	A* object = new(rawMemory) A();

	// delete object;
	// CANNOT CALL delete, it would lead to memory corruption
	object->~A(); // manually call destructor, if you need to get rid of object

	delete[] rawMemory; // at the end, release ALL allocated memory

	return 0;
}
```

There are some circuimstances for using placement `new`. You must not call any kind of `delete` operator on data allocated using placement `new`. It would lead into memory corruption, since default C/C++ allocator (either `new` or `malloc`) keeps track of beginning address of allocated block and size of allocated chunk. Freeing memory with different address would lead into allocator failing to find allocated block.

You also need to keep in mind you need to manage the memory manually byte by byte. So lets say you want to allocate more instances of A object. You have to use pointer arithmetics and move pointer into another address. Here is example:


```c++
A* firstObject = new(rawMemory) A();
// A* secondObject = new (rawMemory) A();
// Cannot do this! It would overwrite the first object
A* secondObject = new(rawMemory + sizeof(A)) A();
// this is the way to go
```

This is the first part of memory allocation and garbage collection implementation series. I would like to continue with articles about memory layout, implementing garbage collector and finalization.
