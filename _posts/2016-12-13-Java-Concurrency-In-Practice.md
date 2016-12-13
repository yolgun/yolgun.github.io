---
title: Java Concurrency In Practice
layout: post
author: yunolgun
permalink: /Java-Concurrency-In-Practice/
source-id: 1dRvLQ1isXVC-cyW-IYF9LV8NbDT9AXgL3hChhNtJ8yE
published: true
---
# Java Concurrency In Practice

## Thread Safety

* Concurrent programming is about *shared, mutable *state.

* **Atomic **operations: Indivisible set of instructions. Common violations are *read-modify-write *and *check-then-act*. Violating it causes *race conditions*.

* **Race conditions **means correctness of the computation depends on relative timing of threads.

* Stateless classes are thread-safe.

* **AtomicLong **and other lock-free thread-safe classes in java.util.concurrent.atomic package can be used to use common compound operations as atomic. For example, AtomicLong::incrementAndGet method

* **Intrinsic locks **are another method to turn compound operations into atomic. **Synchronized **keyword is used for that. Can be used to lock a block of code, on a non-static method(@GuardedBy(this)), a static method (@GuardedBy(this.getClass())). Essentially they act as **mutexes**. 

* Intrinsic locks are **reentrant**. Means a thread con lock on an object more than once. This lets to call synchronized methods of superclasses. This is different than posix mutexes.

* When an object is guarded by synchronization, it is important synchronize all accesses including read-only ones.

* Synchronization is expensive and can reduce performance drastically if used wrong, especially in multi-processor environments. Try to keep synchronized blocks small and non-blocking. 

