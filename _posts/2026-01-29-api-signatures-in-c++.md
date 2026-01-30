---
layout: single
title: "C++ Signatures: It’s Not Just About Performance, It’s About Ownership"
date: 2026-01-29
author_profile: true
author: granza
categories: [Learning, Systems]
tags: [C++, Ownership, Memory Management, API]

sidebar:
  - title: "Further Reading"
    text: |
      **Book:** [Modern Effective C++](https://ananyapam7.github.io/resources/C++/Scott_Meyers_Effective_Modern_C++.pdf)
      **Reference:** [Value categories](https://en.cppreference.com/w/cpp/language/value_category.html)
---

In Java and Python, every object is tracked by a pointer, but the language hides it to make devs' lives easier.
These languages always "pass by value", but since every object is a reference, at the end of the day, the observed behaviour feels like passing "by reference".

If you want to pass something "by value", you need to actively build a deep copy, or call a method that creates a deep copy internally. For these languages, which usually care more about usability than performance, passing "by reference" is a good generic way to do things.

You may be wondering: but passing "by reference" is always better in C++ too! You literally avoid creating a new object. Why would someone need to use a different approach? Well, the devil lies in the details.

Java, Python, and many other languages hide ownership of variables. I mean, if the ownership of an object gets hard to track, who cares? The garbage collector will clean up the mess for us after all!

For people working with C++, or even Rust, languages that delegate ownership management to the developer, passing things by reference can become a nightmare. That’s why things like [smart pointers](https://en.cppreference.com/w/cpp/memory) and [RAII](https://en.cppreference.com/w/cpp/language/raii) were created: to make sure that whoever allocated memory will not forget to deallocate it, or worse, deallocate it twice!

This brief article discusses some rules of thumb to help you not lose track of ownership when writing API signatures.

# Rules of thumb to decide signatures

Before starting, you have to remember that C++ has and cares about rvalues and lvalues:
 - rvalues: Temporary objects that do not have a persistent memory address. E.g., literals and temporary return values.
 - lvalues: Objects that have an identifiable location in memory (an address) and a name. E.g., variables with names. 

```c++
std::println("{}", 1); // prints an rvalue

int x = 2;
std::println("{}", x); // prints an lvalue
```

When writing a method, you usually don't know (or even don't care) if the value passed will be an rvalue or an lvalue.

That’s why these rules of thumb exist: to help you decide which signature to use, without overthinking and while avoiding strange pitfalls or corner cases related to ownership.

As a rule of thumb, there are principles you can follow when developing API signatures.

The main keyword here is **OWNERSHIP**, so you can divide methods into three main categories:
- owning methods
- non-owning methods
- generic methods

## Owning Methods
Here you receive the argument and own it; in this example, you "steal" (move) the data.

```c++
Person temp_person;

void set_name(std::string str){
    temp_person.name = std::move(str);
}

set_name(std::string("Jane Doe"));      // 1

std::string name = "John Doe";
set_name(name);                         // 2

set_name(std::move(name));              // 3
```

For some people this looks weird, even for me when I first heard about this kind of pattern.
You know, always creating a copy? sounds crazy at first. But lets break it down.

The good stuff about passing by value is that you minimize the amount of move/copy operations you do overall.

Let me explain the marks in the code:

1 - When passing an rvalue, the "str" variable (i.e., the param of `set_name`) is created via move construction, and after that it's moved again. So there are two moves here.

2 - When passing an lvalue, the "str" variable (i.e., the param of `set_name`) is created by the copy construction, and after that it's moved. So there are one move and one copy. Unfortunely, it's impossible to do it in fewer operations, since the method does not know if it can take the ownership of an lvalue (which may be used in its source).

3 - Additionaly, if the user wants to allow the moving of the lvalue, it can use `std::move` to cast it to an rvalue and benefit of the possible speedup of case 1.

### Is it not fast enought for you?

If you know this is in the middle of your hot path, or you have measured it ant it is killing your performance, one simple way to fix this is using overloads:

```c++
// accepts lvalue references
void set_name(std::string& str){
    temp_person.name = std::move(str);
}

// accepts rvlaue references
void set_name(std::string&& str){
    temp_person.name = std::move(str);
}
```

In this case you would have only one move operation for the calls, no matter whether you pass an rvalue or an lvalue.
Yet, this comes with more code to maintain and, worst of all, the semantics of the method change.
Every variable passed to `set_name` would give its ownership to the method.
This could be a problem in the following scenario.

```c++
Person temp_person;

void set_name(std::string& str){
    temp_person.name = std::move(str);
}

void set_name(std::string&& str){
    temp_person.name = std::move(str);
}

std::string my_precious_string = "password123";
set_name(my_precious_string); // you just lost the ownership of the string

std::println("My password: {}", my_precious_string); // no longer works properly
```

Thats why passing by copy is usally a good design choice.
And if you ever need more sophisticade approaches, you implement it latter.

For even more generic approaches, you should look at "Modern Effective C++.
The parts talking about Universal References would make you happy, also known as Forwarding References.

### Pipe Methods

This part is not based on the "Modern Effective C++", it is more subjective to my previous experience.

This is a special case of owning methods.
Since it changes an argument, it behaves as a temporary owner.
This is an example to borrowing ownership.

You pass by value, modify and than return.

```c++
std::string add_exclamation(std::string str){
    str.push_back('!');
    return str;
}

std::string phrase = "Lorem Ipsum";
phrase = add_exclamation(phrase);
```

This guarantees that you do not lose the ownership of something you didn't mean to lose.
Another way to do this would be pass by reference, but be aware of losing ownership:
```c++
void add_exclamation(std::string& str){
    str.push_back('!');
}
```

Pass by value has another pro, it is friendly to move-only objects, as [`std::thread`](https://en.cppreference.com/w/cpp/thread/thread) and [`std::unique_ptr`](https://en.cppreference.com/w/cpp/memory/unique_ptr).

```c++
std::unique_ptr<int> foo(std::unique_ptr<int> ptr){ 
    // You are the owner of ptr
    // Modifications DO NOT afect the caller.
    return ptr;
}

void bar(std::unique_ptr<int>& ptr){
    // You are NOT the owner of ptr
    // Modifications afect the caller.
}
```

## Not Owning Methods
In this case, you only receive the value, but you do not plan to modify it.

```c++
void print_name(const std::string& str){
    std::println("Name: {}", str);
}

print_name(std::string("Jane Doe"));    // 1

std::string name = "John Doe";
print_name(name);                       // 2
```

Here are the cases:

1 - When passing an rvalue, it binds to a `const std::string&`, since it is `const`, the language allows binding rvalue. No copies or moves happen here.

2 - When passing an lvalue, no copies or moves happen either.

Great! Next!

## Generic Methods

This is a huge topic, but when you want a method that is extremely generic, use forwarding references, `auto` return types, and perfect forwarding.

But before we start, forwarding references should be reserved for generic wrappers and factory-style utilities. Overusing them in regular APIs often leads to hard-to-debug overload resolution issues.

```c++
// forwarding references only work using templates.
template<class T>
decltype(auto) omega_foo(T&& t){ // it creates references that work for rvalues and lvalues.
    // calls another generic method; forward replicates the r-ness or l-ness of the original param
    return omega_bar(std::forward<T>(t)); // the call is usually to a constructor or to a function overloaded on rvalue and lvalue references
}
```

This pattern is used widely across the STL, but the best example of it is `emplace_back`.

```c++
std::vector<std::pair<int, int>> v;
v.emplace_back(1, 1);           // perfect-forwarded arguments

std::pair<int, int> x = {2, 2};
v.emplace_back(x);              // copy
v.emplace_back(std::move(x));   // move
```
In both calls it is resolved to the right (usually optimized) constructor.

You could implement some stuff to initialize objects:

```c++
template<class T, class U>
auto constructor_wrapper(U&& u){
    // builds and returns an object of type T initialized with u
    return T(std::forward<U>(u)); 
}
```

It's also possible to use Forwarding References with variadic parameters.

```c++
template<class T, class... Args>
auto constructor_wrapper(Args&&... args){
    // builds and returns an object of type T initialized with whatever was passed.
    return T(std::forward<Args>(args)...);
}
```

One common problem with this is the fact that it accepts EVERYTHING.
So if you do not want to accept some types, in C++20 you can use concepts to do it:

```c++
template <typename Arg>
concept ValidArg = !std::is_null_pointer_v<std::remove_cvref_t<Arg>>;

template <typename T, typename... Args>
requires (ValidArg<Args> && ...)
auto constructor_wrapper(Args&&... args) {
    return T(std::forward<Args>(args)...);
}
```
This implementation does not allow `std::nullptr_t`.

## Disclaimer

Things are not as easy as they seem in these examples.
There are many corner cases, and when you need to optimize things, you should design better signatures.
But before designing, measure performance to make sure the changes are actually needed.

To better understand the concepts presented here, as well as their corner cases, read *Modern Effective C++*.

And ALWAYS remember:
> “APIs live longer than optimizations.”