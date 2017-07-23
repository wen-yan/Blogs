---
categories: c++
---

### C++ some special types

##### Pointer of array
```c++
// p is a pointer to int array
// with 3 elements
int (*p)[3];
// equals to
using type = int(*)[3];
type p;

// p is a pointer array with
// 2 elements point to an int array
// with 3 elements
int (*p[2])[3];
// equals to
type p[2];
```
##### Function returns pointer to array
```c++
// function f returns pointer to
// int array with 3 elements
int (*f())[3]
{
}

// using auto
auto f() -> int(*)[3]
{
}

// function pointer
int (*pf())[3];

// function pointer array
int (*(pfa[2])())[3];
```
> **If possible, do not use it. Instead using typedef/using to define types.**
