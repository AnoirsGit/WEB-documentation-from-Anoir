# Demystifying memory management in modern programming languages

## 1. Introduction to Memory management

Memory management is the process of controlling and coordinating the way a software application access **computer memory**.

## Stack

The stack is used for **static memory allocation** and as the name suggests it is a last in firs out(**LIFO**) stack

- **Stack frames** is where the execution data of the functions are stored. Each frame is a block of space where the data required for that func is stored
- **Multi-threaded applications** can have **stack per thread*
- Typical data that are stored on stack are **local variables** (value types or primitives, primitive constants), **pointers** and **function frames**
- This is where your would encounter **stack overflow errors** as sthe size of the stack is limited compared to the Heap
- There is a **limit on the size** of value that can be stroed on the Stack for most languages

## Heap

Is used for **dynamic memory allocation** and unlike stack, the program needs to look up the data in heap using **pointers**

- It is **slower** than stack as the process of looking up data is more involved but it can store more data than the stack
- This is **shared** among threads of an application
- Due to its dynamic nature heap is **tricky to manage** and this is where most of the memory management issues arise from and this is where the automatic memory management solutions from the language kick in.

## Garbage collection(GC)

Automatic management of heap by freeing unused memory allocations

**JVM(Java/Scala/Groovy/Kotlin), JavaScript, C#, Golang, OCaml,** and **Ruby** are some of the languages that use Garbage collection for memory management by default.

![AZaR0LP](https://user-images.githubusercontent.com/49281851/182045701-1576b88f-6288-4be1-ad00-6d53a24bcc54.gif)

- **MARK & SWEEP GC**: Also known as Tracing GC. Its generally a two-phase algorithm that first marks objects that are still being referenced as "alive" and in the next phase frees the memory of objects that are not alive.

- **Reference counting GC**: In this approach, every object gets a ref count which is incremented or decremented as ref to it change and garbage collection is done when the count becomes zero

## Resource Acquisition is Initialization (RAII)

In this type memory management, an object's memory allocation is tied to its lifetime, which is from construction until destruction. It was introduced in **C++** and used also in **Rust**

## Automatic Reference Counting (ARC)

It's similar to Reference counting GC but instead of running a runtime process at a specific interval the **retain** and **release** instructions are inserted to the compiled code at compile-time and when an object reference becomes zero its cleared automatically as part of execution without any program pause. It also cannot handle cyclic references and relies on the developer th handle that by using cerain keywords. Its a feature of the Clang compiler and provides ARC for Objective C & Swift

## Ownership

It combines RAII with an ownership model, any value must have a variable as its owner(and only one owner at a time) when the owner goes out of scope the value will be dropped freeing the memory regardless of it being in stack or heap memory
