---
layout: single
title: "Atomicity against Data Races"
date: 2026-01-26
author_profile: true
author: granza
categories: [Learning, Systems]
tags: [C++, Concurrency, Atomics]

sidebar:
  - title: "Further Reading"
    text: |
      **Book:** [C++ Concurrency in Action](https://www.manning.com/books/c-plus-plus-concurrency-in-action-second-edition)
      
      **Reference:** [std::atomic docs](https://en.cppreference.com/w/cpp/atomic/atomic)
      
      **Watch:** [Atomic Weapons (CppCon)](https://www.youtube.com/watch?v=A8eCGOqgvH4)
---


I was reading *[C++ Concurrency in Action](https://github.com/samuel-24276/Code-of-CPP-Concurrency-In-Action/blob/master/C%2B%2B%20Concurrency%20in%20Action%20by%20Anthony%20Williams%20(z-lib.org).pdf)* and finally internalized the power of atomicity against data races.

**Atomicity** literally means *inseparable*.

But in concurrency, it is a **semantic guarantee**.

It means that an operation, as *intended by the programmer*, either happens **entirely** or **not at all**, as observed by other threads.

> **Atomicity (the concept, not just `std::atomic`)<br>is the real tool for eliminating data races.**

Mutexes, atomics, and transactions are merely different mechanisms to enforce this same fundamental property.

Let’s take a look at that carefully.

---

### A "Benign" race condition

```cpp
int x = 0;

auto func1 = [&x]() {
    x = 1;
};

auto func2 = [&x]() {
    x = 2;
};

std::jthread t1{func1}, t2{func2};
```

This program has a **data race**, and its behavior is **undefined** *according to the C++ standard*.

However, *conceptually*, this race is often described as "benign":

* Every write is well-formed
* At some point `x` becomes `1`
* At some point `x` becomes `2`
* The order is nondeterministic

Mathematically the expected value of `x` is `1.5` (assuming equal odds), but the program itself is ill-formed by the standard.

This example is useful because it shows **nondeterminism without logical corruption**.

---

### A *bad* race condition: lost updates

Now consider incrementing (`x +=`) instead of assigning (`x =`):

```cpp
int x = 0;

auto func1 = [&x]() {
    x += 1; // increments 1
};

auto func2 = [&x]() {
    x += 2; // increments 2
};

std::jthread t1{func1}, t2{func2};
```

This is still **undefined behavior**, but now the **intended semantics can be violated**.

Why?

Because `x +=` is **not atomic**. It expands to three CPU steps:

1. read `x`
2. modify (add)
3. write `x`

See how this can mess up the order of operations for the two threads:

```text
    Thread 1             Thread 2          Value of X
----------------    ----------------    ----------------
    read (0)                                   0
    add  (1)                                   0
                        read (0)               0
    write(1)                                   1
                        add  (2)               1
                        write(2)               2
```

Here, the increment by `thread 1` is **lost**.
This **is a data race**.


### What we *actually* wanted

We didn’t want a sequence of three steps, we wanted **one indivisible operation**:

- { read + add + write }

If increment were atomic, then both updates would ALWAYS be applied.

---

## Enforcing Atomicity

Here are the simplest ways to implement atomicity.
Keep in mind that while concurrency is a deep topic, these fundamental tools are your first line of defense.

### Using a mutex (the classic way)

```cpp
std::mutex mt;
int x = 0;

auto func1 = [&]() {
    std::lock_guard<std::mutex> lg(mt); // lock the mutex
    x += 1;
    // the lock_guard automatically unlocks the mutex
};

auto func2 = [&]() {
    std::lock_guard<std::mutex> lg(mt); // lock the mutex
    x += 2;
    // the lock_guard automatically unlocks the mutex
};

std::jthread t1{func1}, t2{func2};
```

This works because the critical section is serialized. Only one thread can perform the 3-step sequence at a time.

---

### Using `std::atomic`

```cpp
std::atomic<int> x = 0;

auto func1 = [&]() {
    x += 1; // atomic read-modify-write
};

auto func2 = [&]() {
    x += 2; // atomic read-modify-write
};

std::jthread t1{func1}, t2{func2};
```

Here, `x += n` becomes a **single atomic read-modify-write operation**.

`std::atomic<int>` supports increment as a atomic operation, so no data races.

The final value of `x` is **well-defined** and always `3`.

Be aware that atomics are **not magical**, they do eliminate data races, but do **not** eliminate nondeterminism automatically:

```cpp
std::atomic<int> x = 0;

auto func1 = [&]() {
    x.store(1); // atomically store data
};

auto func2 = [&]() {
    x.store(2); // atomically store data
};

std::jthread t1{func1}, t2{func2};
```

Even being a well-defined and atomic program the final value of `x` is **nondeterministic**

It will be either `1` or `2`, depending on which store happens last.

---

## Takeaway
- **Atomicity is the foundation of correct concurrent reasoning.**
- Everything else is an implementation detail.
