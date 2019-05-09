---
published: false
---
## Memory layout

In this blog post I would like to describe memory layout and memory alignment and explain, why it is important.

## Memory pages
In modern computers memory is divided into chunks called pages. On most systems, page is usually around 4 KiB. When you need to allocate some memory, operating system will allocate it to you page by page. Allocated  memory is usally rounded to whole pages, so allocating 5 KiB of memory would (mostly) end up allocating 2 pages, 8 KiB in total. It is because from CPU point of view, working with continuous memory blocks is very efficient. It can be loaded from RAM in fewer round trips and also it can be stored in CPU caches like L2.

## Words

Word is the smallest amount of memory CPU can work with. On x86 systems it is usally 32 bits, on x64 systems 64 bits. When you need to work with smaller data types (char for example) CPU will have to expand it to its default word size, execute the operation and truncate the result.

## Memory alignment
Most of modern C++ compilers keep data structures memory aligned to multiples of word size.

Does the order of data members in struct (class) matter? The answer is yes, it does. Some compilers might reallign it during compilation, but I wouldn't count on that, because it could lead into compiler-dependent behavior.

Lets see the difference

```c++
struct A
{
	char a;
    int b;
    char c;
}

struct B
{
	int b;
    char a;
    char c;
}

struct C
{
	int b;
    char array[4];
}

int main() {
	cout << "Size of A is: " << sizeof(A) << " bytes" << endl;
	cout << "Size of B is: " << sizeof(B) << " bytes" << endl;
	cout << "Size of C is: " << sizeof(C) << " bytes" << endl;
}
```
Answer is

```
Size of A is: 12 bytes
Size of B is: 8 bytes
Size of C is: 8 bytes
```

as you can see, you can fit two more chars into structure while keeping its memory size, when you align it properly! Lets see how it looks like in memory:

// ADD SCREENSHOTS FROM VS DEBUGGER

// ADD SOME SIMPLE DIAGRAM
