---
title: Effective Modern C++
category: C++
tags: C++
layout: post
---

##### Item 1: Understand template type deduction

> * During template type deduction, arguments that are references are treated as non-references, i.e., their reference-ness ignored.
> * When deducing types for universal reference parameters, lvalue arguments get special treatment.
> * When deducing types for by-value parameters, const and/or volatile arguments are treated as non-const and non-volatile.
> * During template type deduction, arguments that are array or function names decay to pointers, unless they're used to initialize references.

```c++
template <typename T>
void f(ParamType param);

int x = 27;
const int cx = x;
const int& rx = x;
const int* px = &x;

// Case 1: ParamType is a Reference or Pointer, but not a Universal Reference
// reference
template <typename T>
void f(T& param);

f(x);   // T =>       int, param =>       int&
f(cx);  // T => const int, param => const int&
f(rx);  // T => const int, param => const int&

// const reference
template <typename T>
void f(const T& param);

f(x);   // T => int, param =>       int&
f(cx);  // T => int, param => const int&
f(rx);  // T => int, param => const int&

// pointer
template <typename T>
void f(T* param);

f(&x);  // T =>       int, param =>       int*
f(px);  // T => const int, param => const int*

// Case 2: ParamType is a Universal Reference
template <typename T>
void f(T&& param);

f(x);   // T =>       int&, param =>       int&
f(cx);  // T => const int&, param => const int&
f(rx);  // T => const int&, param => const int&
f(27);  // T =>       int , param =>       int&&

// Case 3: ParamType is Neither a Pointer nor a Reference
//  1. ignore reference-ness
//  2. ignore const and volatile
template <typename T>
void f(T param);

f(x);   // T => int, param => int
f(cx);  // T => int, param => int
f(rx);  // T => int, param => int

const char* const ptr = "Fun with pointers";
f(ptr); // T => const char*, param => const char*

// Array Arguments
const char name[] = "J. P. Briggs"; // const char[13]
const char* ptrToName = name;

f(name);    // T => const char*, param => const char*

// Array Arguments by Reference
template <typename T>
void f(T& param);

f(name);    // T => const char [13], param => const char (&)[13]

// Function Arguments
void someFunc(int, double);

template <typename T>
void f1(T param);

tempalte <tyepname T>
void f2(T& param);

f1(someFunc);   // param => void (*)(int, double)
f2(someFunc);   // param => void (&)(int, double)
```

##### Item 2: Understand **auto** type deduction
> * auto type deduction is usually the same as template type deduction, but auto type deduction assumes that a braced initializer represents a *std::initializer_list*, and template type deduction doesn't.
> * auto in a function return type or a lambda parameter implies template type deduction not auto type deduction.

```c++
// auto with braced initializer
auto x = {27};  // x => std:initializer_list<int>, value is {27}
auto x{27};     // x => std:initializer_list<int>, value is {27}

// template can't deduce initializer_list
template <typename T>
void f(T param);

f({11, 23, 9});     // error! can't deduce type for T

// Need to use initializer_list directly
template <typename T>
void f(std::initializer_list<T> param);

f({11, 23, 9});     // T => int
```