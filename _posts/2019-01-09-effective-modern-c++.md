---
title: Effective Modern C++
category: C++
tags: C++
layout: post
---

#### Item 1: Understand template type deduction

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

#### Item 2: Understand **auto** type deduction
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

// Can't deduc auto when it's using as lambda argument or return type
auto f() { return { 1, 2, 3 }; }  // can't deduce type for { 1, 2, 3 }
auto f = [](auto arg){...};
f({1, 2, 3});   // can't deduce type for { 1, 2, 3 }
```

#### Item 3: Understand **decltype**

> * decltype almost always yields the type of a variable or expression without any modifications.
> * For lvalue expressions of type T other than names, decltype always reports a type of T&.
> * C++14 supports decltype(auto), which, like auto, deduces a type from its initializer, but it performs the type deduction using the decltype rules.

```c++
const int i = 0;            // decltype(i) => const int
bool f(const Widget& w);    // decltype(w) => const Widget&
                            // decltype(f(w)) => bool
struct Point { int x, y; }; // decltype(Point::x) => int
Widget w;                   // decltype(w) => Widget
vector<int> v;              // decltype(v) => vector<int>
                            // decltype(v[0]) => int&

// decltype(auto)
const Widget& cw;
decltype(auto) cw2 = cw;    // decltype(auto) => const Widget&

// deduce reference
int i = 10;                 // decltype(i) => int
                            // decltype((i)) => int&

decltype(auto) f() { static int x = 0; return x; }      // decltype(auto) => int;
decltype(auto) f() { static int x = 0; return (x); }    // decltype(auto) => int&;
```

#### Item 4: Know how to view deduced types

> * Deduced types can often be seen using IDE editors, compiler error messages, and the Boost TypeIndex library.
> * The results of some tools may be neither helpful nor accurate, so an understanding of C++'s type deduction rules remains essential

#### Item 5: Prefere auto to explicit type declarations

> * auto variables must be initialized, are generally immune to type mismatches that can lead to portability or efficiency problems, can ease the process of refactoring, and typically require less typing than variables with explicitly specified types.
> * auto-typed variables are subject to the pitfalls described in Items 2 and 6.

```c++
// use auto for better performance
std::unordered_map<std::string, int> m;

// actually it's std::pair<const std::string, int> in m,
// not std::pair<std::string, int>, so a temporary
// std::pair<const std::string, int> is created and
// conversion is needed !!! temporary p is destroyed
// before leaving the for scope
for (const std::pair<std::string, int>& p : m)
{                                               
    // ...                                      
}                                               

// no conversion or temporary object
for (const auto& p : m) {}                      
```

#### Item 6: Use the explicitly typed initializer idiom when auto deduces undesired types

> * "Invisible" proxy types can cause auto to deduce the "wrong" type for an initializing expression.
> * The explicitly typed initializer idiom forces auto to deduce the type you want it to have.

```c++
// problem when using auto with "invisible" proxy
std::vector<bool> features(const Widget& w);
auto flag = features(w)[5];     // a proxy references temporary vector<bool> is returned.
func(flag);                     // temporary vector<bool> is destroyed !!!

// solution, cast
auto flag = static_cast<bool>(features(w)[5]);
```

#### Item 7: Distinguish between () and {} when creating objects

> * Braced initialization is the most widely usable initialization symtax, it prevents narrowing conversions, and it's immune to C++'s most vexing parse.
> * During constructor overload resolution, braced initializers are matched to std::initializer_list parameters if at all possible, even if other constructors offer seemingly better matches
> * An example of where the choice between parentheses and braces can make a significant difference is creating a std::vector<numeric type> with two arguments.
> * Choosing between parentheses and braces for object creation inside templates can be challenging.

```c++
class Widget
{
public:
    Widget();
    Widget(int i, bool b);
    Widget(int i, double d);
    Widget(std::initializer_list<bool> il);
};
Widget w1(10, true);    // calls second ctor
Widget w2{10, true};    // calls std::initializer_list ctor, but error! requires narrowing conversions
Widget w3{};            // calls first ctor, but not the initializer_list one. because it's empty braces
Widget w4({});          // calls initializer_list one, with empty list
Widget w5{ {} };          // same as above

class Widget
{
public:
    Widget(int i, bool b);
    Widget(int i, double d);
    Widget(std::initializer_list<std::string> il);
};
Widget w2{10, true};    // calls first ctor, because 10 and true can't be converted to std::string
```

#### Item 8: Prefer nullptr to 0 and NULL

> * Prefer nullptr to 0 and NULL
> * Avoid overloading on integral and pointer types

#### Item 9: Prefer alias declarations to typedefs

> * typedefs don't support templatization, but alias declarations do.
> * Alias templates avoid the "::type" suffix and, in templates, the "typename" prefix often required to refere to typedefs.
> * C++14 offers alias templates for all teh C++11 type traits transformations.

#### Item 10: Prefer scoped snums to unscoped enums

> * C++98-style enums are no known as unscoped enums.
> * Enumerators of scoped enums are visible only within the enum. They convert to other types only with a cast.
> * Both scoped and unscoped enums support specification of the underlying type. The default underlying type for scoped enums is int. Unscoped enums have no default underlying type.
> * Scoped enums may always be forward-declared. Unscoped enums may be forward-declared only if their declaration specifies and underlying type.

#### Item 11: Prefer deleted functions to private undefined ones

> * Prefer deleted functions to private undefined ones.
> * Any function may be deleted, including non-member functions and template instantiations.

```c++
// any function can be deleted
void test_only_int(int value);
void test_only_int(bool) = delete;
void test_only_int(char) = delete;

test_only_int(true);    // error! deleted
test_only_int('c');     // error! deleted

// delete template function
template <typename T>
void processPointer(T* p);

template <>
void processPointer<void>(void*) = delete;

template <>
void processPointer<char>(char*) = delete;
```

#### Item 12: Declare overriding functions override

> * Declare overriding functions override.
> * Member function reference qualifiers make it possible to treat lvalue and rvalue objects (*this) differently.

```c++
class Widget
{
public:
    using DataType = std:;vector<double>;
    DataType& data() & { return values; }               // for lvalue Widgets
    DataType&& data() && { return std::move(values); }  // for rvalue Widgets
private:
    DataType values;
};
auto vals1 = w.data();              // call lvalue overload
auto vals2 = makeWidget().data();   // call rvalue overload
```