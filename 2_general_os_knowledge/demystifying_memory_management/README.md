# Demystifying memory management in modern programming languages

## 1. Introduction to Memoru management

Memory management is the process of controlling and coordinating the way a software application access **computer memory**.

Whe a software runs on a target OS on a Computer it needs access to the computer **RAM** to: 
  - load its own **bytecode** that needs to be executed
  - store the **data values** and **data structures** used by the program that is executed
  - load any **run-time systems** that are required for the program to execute

When a software program uses memory there are two regions of memory they use, apart form the space used to load the bytecode, **STACK and HEAP** memory

## Stack

The stack is used for **static memory allocation** and as the name suggests it is a last in firs out(**LIFO**) stack 
   - Due to this nature, the process of storing and retrieving data from the stack is **very fast** as there is no lookup required, you just store and retrieve data from the topmost block on it.
   - But this means any data that is stored on the stack has to be **finite and static** (The size of the data is known at compile-time).
   - This is where the execution data of the functions are stored **stack frames**. Each frame is a block of space where the data required for that func is stored. For example, ecery time a function declared a new variable, it is "pushed" onto the topmost block in the stack. Then every time function exits, the topmost block is cleared. These scan be determined at compile time due to the static natrure of the data stored here.
   - **Multi-threaded applications** can have **stack per thread**
   - Memory management of the stack is simple and straightforward and is done by the OS
   - Typical data that are stored on stack are **local variables** (value types or primitives, primitive constants), **pointers** and **function frames**
   - This is where your would encounter **stack overflow errors** as sthe size of the stack is limited compared to the Heap
   - There is a **limit on the size** of value that can be stroed on the Stack for most languages

Stack used in JavaScript, objects are stored in Heap and referenced when needed. Here is a [video](https://www.youtube.com/watch?v=95_CAUC9nvE) of the same

## Heap

Is used for **dynamic memory allocation** and unlike stack, the program needs to look up the data in heap using **pointers**

  - It is **slower** than stack as the process of looking up data is more involved but it can store more data than the stack
  - This is **shared** among threads of an application
  - Due to its dynamic nature heap is **tickire to manage** and this is where most of the memory management issues arise from and this is where the automatic memory management solutions from the language kick in.
  - Typical data that are stored on the heap are **global variables, reference typed** like objects, strings, maps, and other complex data structures
  - This is where you would encounter **out memory errors** if your application tries to use more memory than the allocated heap
  - Generally, there is **no limit** on the size of the value that can be stored on the heap. Of course, there is the upper limit of how much memory is allocated to the application.


## Manual memory management

The language doesn't manage memory for you by default, it's up to you to allocate and free memory for the objects you create. For example, **C** and **C++**. They provide the **malloc, realloc, calloc,** and **free** methods to manage memory and it's up to the developer to allocate and free heap memory in the program and make use of pointers efficiently to manage memory. Let's just say that it's not for everyone 

## Garbage collection(GC)

Automatic management of heap by freeing unused memory allocations. GC is one of the most common memory management in modern languages and the process often runs at certain intervals and thus might cause a minor overhead called pause times.

**JVM(Java/Scala/Groovy/Kotlin), JavaScript, C#, Golang, OCaml,** and **Ruby** are some of the languages that use Garbage collection for memory management by default.

![image](https://user-images.githubusercontent.com/49281851/182045648-378ea1dc-7a57-4325-ac30-78436de9452b.png)

  - **MARK & SWEEP GC**: Also known as Tracing GC. Its generally a two-phase algorithm that first marks objects that are still being referenced as "alive" and in the next phase frees the memory of objects that are not alive. **JVM, C#, JS** and **Golang** employ this approach for example. In JVM there different GC algorithms to choose from while JS engines like V8 use a MARK § SWEEP GC  along with Reference counting GC to compliment it. 

  - **Reference counting GC**: In this approach, every object gets a ref count which is incremented or decremented as ref to it change and garbage collection is done when the count becomes zero. It's not very preffered as it cannot handle cyclic references.

## Resource Acquisition is Initialization (RAII)

In this type memory management, an object's memory allocation is tied to its lifetime, which is from construction until destruction. It was introduced in **C++** and used also in **Rust**

## Automatic Reference Counting (ARC)

It's similar to Reference counting GC but instead of running a runtime process at a specific interval the **retain** and **release** instructions are inserted to the compiled code at compile-time and when an object reference becomes zero its cleared automatically as part of execution without any program pause. It also cannot handle cyclic references and relies on the developer th handle that by using cerain keywords. Its a feature of the Clang compiler and provides ARC for Objective C & Swift

## Ownership

It combines RAII with an ownership model, any value must have a variable as its owner(and only one owner at a time) when the owner goes out of scope the value will be dropped freeing the memory regardless of it being in stack or heap memory. It is kind of like Compile-time reference counting. It is used by Rust, in my research I couldn't find any other language using this exact mechanism.
