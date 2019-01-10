---
title: Some special type declarations of C++
excerpt: Some C++ type decalrations are not frequently used.
category: C++
tags: C++
layout: post
---

C++ has some special type declarations. Especially combine pointer, reference, array and function pointer. Sometimes it's hard to understand what it is.

In following samples, `typeid(x).name()` is used to make it more clear, because it uses a more clear name which might be easily understood than  literal descriptions.
> Note : `typeid(x).name()` might return different names when using different compilers. My compiler is *g++ (GCC) 6.2.1 20160916 (Red Hat 6.2.1-3)*

##### array and pointer
```c++
// a is a 3 elements array
// typeid(a) = A3_i
int a[3];

// b is a 3 elements (pointer) array
// typeid(b) = A3_Pi
int* b[3] = { &a[0], &a[1], &a[2] };

// c is a pointer to 3 elements array
// typeid(c) = PA3_i
int (*c)[3] = &a;

// d is an array has 2 pointer elements which points to a 3 elements array
// typeid(d) = A2_PA3_i
int (*d[2])[3] = { &a, &a };
```

##### array and reference
Reference is similar to pointer except what `typeid(x).name()` returns.
```c++
// a is a 3 elements array
// typeid(a) = A3_i
int a[3];

// b is a reference to a 3 elements array
// typeid(b) = A3_i,  same as a
int (&b)[3] = a;
```
> Note : reference array is illegal. So there is no sample for `int& b[3]` and `int (&b[2])[3]`.

##### array and function
```c++
// a is a 3 elements array
// typeid(a) = A3_i
int a[3];

// function b returns pointer to a 3 elements array
// typeid(b) = FPA3_ivE
int (*b())[3] { return &a; }
// or auto style
auto b() -> int (*)[3] { return &a; }

// function c returns reference to a 3 elements array
// typeid(c) = FPA3_ivE
int (&c())[3] { return a; }
// or auto style
auto c() -> int (&)[3] { return a; }

// pf is a pointer of function which returns pointer to a 3 elements array
// typeid(pf) = PFPA3_ivE
int (*(*pf)())[3] = &b;

// pfa is a 2 elements array whose elements are pointers of function which returns pointer to a 3 elements array
// typeid(pfa) = A2_PFPA3_ivE
int (*(*pfa[2])())[3] = { &b, &b };
```
> Note : if possible, do not use so complex declarations. Instead using typedef/using to define types.

```c++
using array_pointer = int (*)[3];
using func_array_pointer = array_pointer (*)();
func_array_pointer pf = &b;
func_array_pointer pfa[2] = { &b, &b };
```