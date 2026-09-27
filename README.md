# Comprehensive Guide: Core Python Fundamentals, Memory Architecture & Systems Concepts

> **Academic Reference:** Programming Fundamentals to Object-Oriented Programming (OOP) in Python  
> **Topic Focus:** Low-level memory layout, language semantics, concurrent programming primitives, distributed consistency, and computer systems fundamentals.

---

## Table of Contents
1. [Python Execution Model & Architecture](#1-python-execution-model--architecture)
2. [Data Mutability, Immutability & Reference Semantics](#2-data-mutability-immutability--reference-semantics)
3. [Memory Layout: The Stack, The Heap, and Value Storage](#3-memory-layout-the-stack-the-heap-and-value-storage)
4. [Variable Scope, Namespaces, and the LEGB Rule](#4-variable-scope-namespaces-and-the-legb-rule)
5. [Functions as First-Class Citizens & Execution Contexts](#5-functions-as-first-class-citizens--execution-contexts)
6. [Memory Management & Automatic Garbage Collection](#6-memory-management--automatic-garbage-collection)
7. [Concurrency Primitives: Shared Memory, Race Conditions & Mutex Locks](#7-concurrency-primitives-shared-memory-race-conditions--mutex-locks)
8. [Data Consistency vs. Inconsistency in Systems](#8-data-consistency-vs-inconsistency-in-systems)
9. [Scalability: Horizontal, Vertical & Concurrency Bottlenecks](#9-scalability-horizontal-vertical--concurrency-bottlenecks)
10. [Database Management Systems & Key-Value Stores](#10-database-management-systems--key-value-stores)
11. [Comparative Language Ecosystems: Picking the Right Tool](#11-comparative-language-ecosystems-picking-the-right-tool)
12. [Computer Systems Architecture: Introduction to RISC-V](#12-computer-systems-architecture-introduction-to-risc-v)
13. [Synthesis: Bridge to Object-Oriented Programming (OOP)](#13-synthesis-bridge-to-object-oriented-programming-oop)

---

## 1. Python Execution Model & Architecture

Python is an interpreted, high-level, dynamically typed language primarily executed via **CPython** (the standard C reference implementation). When executing a `.py` script:
1. **Compilation to Bytecode:** The source text is parsed into an Abstract Syntax Tree (AST) and emitted as intermediate bytecode stored in `.pyc` files (`__pycache__`).
2. **Virtual Machine Interpretation:** The **Python Virtual Machine (PVM)** is a stack-based runtime loop executing intermediate bytecode operations sequentially.
3. **Dynamic Typing:** Type verification occurs at runtime. Identifiers do not hold fixed memory slots; they act as symbolic labels pointing to objects allocated in heap space.

---

## 2. Data Mutability, Immutability & Reference Semantics

In Python, all entities are objects (`PyObject`) defined by three core properties:
- **Identity (`id()`):** The unique integer address representing the object's physical memory location.
- **Type (`type()`):** The class interface dictating valid operations and behaviors.
- **Value:** The underlying data buffer or payload.

### 2.1 The Concept of References
Variables do not store raw data values directly on the stack. Instead, assigning a variable creates a binding between an identifier name in a namespace and a target object residing on the heap.

```python
# Reference aliasing vs separate allocation
a = [10, 20, 30]
b = a  # 'b' points to the exact same heap memory allocation

print(f"Address of a: {hex(id(a))}")
print(f"Address of b: {hex(id(b))}")
print(f"Identity equality: {a is b}")  # True

b.append(40)
print(f"Value of a after mutating b: {a}")  # [10, 20, 30, 40]
2.2 Immutable Data Types
An object is immutable if its internal state cannot be modified after instantiation.
Built-in types: int, float, bool, str, tuple, frozenset, bytes.
Any modification operation allocates an entirely new object on the heap and rebinds the pointer.
num = 500
initial_id = id(num)
num += 1

print(f"Same object? {id(num) == initial_id}")  # False (New PyObject created)
Optimization: Integer Interning
CPython optimizes small scalar allocations by pre-allocating an internal global array for integers between -5 and 256. Variables assigned values in this range bind to the same singleton objects.
2.3 Mutable Data Types
An object is mutable if its internal data buffer can be updated in-place without altering its memory identity (id()).
Built-in types: list, dict, set, bytearray, custom class instances.
items = ["first", "second"]
addr = id(items)
items[0] = "updated"

print(f"Address preserved: {id(items) == addr}")  # True
2.4 Pitfall: Mutable Default Arguments
Default argument expressions are evaluated once at function definition time, not at each call. Passing mutable types as default arguments causes state to persist across calls.
# ANTI-PATTERN:
def register_user(username, registry=[]):
    registry.append(username)
    return registry

# CORRECT IDIOMATIC PATTERN:
def register_user_safe(username, registry=None):
    if registry is None:
        registry = []
    registry.append(username)
    return registry
3. Memory Layout: The Stack, The Heap, and Value Storage
+-------------------------------------------------------------+
|                      Virtual Memory Space                   |
+-------------------------------------------------------------+
|  STACK MEMORY                                               |
|  - Function Call Frames (Activation Records)                |
|  - Frame pointers, return program counters                  |
|  - Local variable references (pointers to Heap)             |
|  - Strict LIFO allocation / automatic deallocation         |
+-------------------------------------------------------------+
|                            | (Grows downwards)              |
|                            v                                |
|                            ^                                |
|                            | (Grows upwards)                |
+-------------------------------------------------------------+
|  HEAP MEMORY                                                |
|  - Dynamic allocations (`PyObject`, dictionaries, lists)    |
|  - Managed via Python Memory Manager (pymalloc + system)    |
|  - Object headers, reference counts, type descriptors       |
|  - Monitored by the Garbage Collector                      |
+-------------------------------------------------------------
