# 🚀 Visualizing memory management in V8 Engine (JavaScript, NodeJS, Deno, WebAssembly)

## V8 memory structure

Since JS is single-threaded V8 also uses a single process per JS context, and workers spawns new process. A running program is always represented by some allocated memory in the V8 process and this is called **Resident Set**

![image](https://user-images.githubusercontent.com/49281851/182832569-2db02901-5041-4f32-ad78-caef0ed9db13.png)

## Heap Memory

This is where V8 stores objects or dynamic data. This is biggest block of memory area and this is where **Garbage Collector** takes place. The entire heap memory is not garbage collected, only Young and Old space managed by garbage collection. Heap is further divided into below:

- **New Space**: New space or **"Young generation"** is where now objects live and most of these objects are shor-lived. This space is small and has two **semi-space**. This space is managed by the **"Scavanger(Minor GC)"**

- **Old Space**: Old space or **"Old generation"** is where objects that is survived the "New space" for two minor GC cycles are moved to. This space is managed up by the **"Major GC(Mark-Sweep & Mark-Compact)"**.

  - **Old pointer space**: Contains survived objects that have pointers to other objects.
  - **Old data space**: Contains objects that just contain data(no pointer other objects). Strings, boxed numbers, and arrays of unboxed doubles are moved here after surviving in "New space" for two minor GC cycles.

- **Large object space**: This is where are larger that the size limits of other spaces live
- **Code-space**: This is where the **Just in Time(JIT)** compiler stores compiled code Blocks. This is the only space with executable memory(although `Codes` may be allocated in "Large object space", and those are executable, too)

- **Cell space, property cell space, and map space**: These spaces contain `Cells, PropertyCells` and `Maps`, respectively. Each of these spaces contains objects which are all the same size and has some constraints on what kind of objects they point to, which simplifies collection.

## V8 memory usage (Stack vs Heap)

[this is demonstration of how does it works](https://speakerdeck.com/deepu105/v8-memory-usage-stack-and-heap)

- **Global scope** is kept in a "Global frame" on the Stack
- Every function call is added to the stack memory as frame-block
- All local variables including args and the return valus is saved within the function frame-block on the Stack
- All primitive types are stored directly on the Stack. This applies to global scope as well
- All object types are created on the Heap and is referenced from the Stack using Stack pointers. Functions are just objects in JS. Also applies to global scope as well
- Functions called from the current function is pushed on top of the Stack
- When a function returns its frame is removed from the Stack
- Once the main process is complete, the objects on the Heap do not have any more pointers from Stack and becomes orphan
- Unless you make a copy exlicitly, all object references within other objects are done using reference pointers

## V8 Memory management: Garbage collection

V8 manages the heap memory be garbage collection.

> **Orinoco** is the codename of the V8 GC project to make use of parallel, incremental and concurrent techniques for garbage collection, to free the main thread

### Minor GC (Scavenger)

This type of GC keeps the young or new generation space compact and clean. Objects are allocated in new-space, which is fairly small (between 1 and 8 MB, depending on behavior heuristics).

Allocation in "new space" is very cheap: there is an allocation pointer which we increment whenever we want to reserve space for a new object. When the allocation pointer reaches the end of the new space, a minor GC is triggered. This process is also called **Scavenger** and it implements [Cheney's algorithm](https://en.wikipedia.org/wiki/Cheney's_algorithm). It occurs frequently and uses parallel helper threads and is very fast.
[demonstration](https://speakerdeck.com/deepu105/v8-minor-gc)

### Major GC

This type of GC keeps the old generation space compact and clean. This is triggered when V8 decides there is not enough old space, based on a dynamically computed limit, as it gets filled up from minor GC cycles. GC is done using the **Mark-Sweep-Compact** algorithm. It uses a **tri-color**(white-grey-black) marking system. Hence major GC is a three-step process and the third step is executed depending on a fragmentation heuristic.
![image](https://res.cloudinary.com/practicaldev/image/fetch/s--FJzf91b6--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://i.imgur.com/rcjSZ0T.gif)

- **Marking**: First step, common for both algorithms, where the garbage collector identifies which objects are in use and which ones are not in use. The objects in use or reachable from GC roots(Stack pointers) recursively are marked as alive. It's technically a depth-first-search of the heap which can be considered as a directed graph
- **Sweeping**: The garbage collector traverses the heap and makes note of the memory address of any object that is not marked alive. This space is now marked as free in the free list and can be used to store other objects
- **Compacting**: After sweeping, if required, all the survived objects will be moved to be together. This will decrease fragmentation and increase the performance of allocation of memory to newer objects

This type of GC is also referred to as stop-the-world GC as they introduce pause-times in the process while performing GC. To avoid this V8 uses techniques like

![image](https://user-images.githubusercontent.com/49281851/182500586-aff93f29-470b-4304-a8dc-565d23dec0de.png)

- **Incremental GC**: GC is done in multiple incremental steps instead of one.
- **Concurrent marking**: Marking is done concurrently using multiple helper threads without affecting the main JavaScript thread. Write barriers are used to keep track of new references between objects that JavaScript creates while the helpers are marking concurrently.
- **Concurrent sweeping/compacting**: Sweeping and compacting are done in helper threads concurrently without affecting the main JavaScript thread.
  Lazy sweeping. Lazy sweeping involves delaying the deletion of garbage in pages until memory is required.
