---
title: c++ template tips
category: C++
tags: C++ template
layout: post
---

##### char array reference argument
```c++
tempalte <std::size_t N>
constexpr std::size_t get_length(const char (&p)[N]) noexcept
{
    return N;
}

std::size_t length = get_length("hello world");
```