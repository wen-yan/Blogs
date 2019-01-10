---
title: Effective Modern C++
category: C++
tags: C++
layout: post
---

##### Item 1: Understand template type deduction

> * During template type deduction, arguments that are references are treated as non-references, i.e., their reference-ness ig ignored.
> * When deducing types for universal reference parameters, lvalue arguments get special treatment.
> * When deducing types for by-value parameters, const and/or volatile arguments are treated as non-const and non-volatile.
> * Duringtemplate type deduction, arguments that are array or function names decay to pointers, unless they're used to initialize references.

```C++

```