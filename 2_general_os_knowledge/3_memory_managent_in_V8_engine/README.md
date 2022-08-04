# 🚀 Visualizing memory management in V8 Engine (JavaScript, NodeJS, Deno, WebAssembly)

## V8 memory structure

Since JS is single-threaded V8 also uses a single process per JS context and hence if you use service workers it will spawn a new V8 process per worker. A running program is always represented by some allocated memory in the V8 process and this is called **Resident Set**. This is further divided into different segments as:

![image](https://user-images.githubusercontent.com/49281851/182832569-2db02901-5041-4f32-ad78-caef0ed9db13.png)

## Heap Memory

This is where V8 stores objects or dynamic data. This is biggest block of memory area and this is where **Garbage Collector** takes place. The entire heap memory is not garbage collected, only Young and Old space managed by garbage collection. Heap is further divided into below:

- **New Space**: New space or **"Young generation"** is where now objects live and most of these objects are shor-lived. This space is small and has two **semi-space**, similar to **SO** & **S1** in JVM. This space is managed by the **"Scavanger(Minor GC)"**, we will look at it later. The size of the new space can be controlled using the `--min_semi_space_size`(Initial) and `--max_semi_space_size`(Max) V8 flags.

- **Old Space**: Old space or **"Old generation"** is where objects that is survived the "New space" for two minor GC cycles are moved to. This space is managed up by the **"Major GC(Mark-Sweep & Mark-Compact)"**. The size of old space can be controlled using the `--initial_old_space_size`(Initial) and `--max_old_space_size`(Max) V8 flags. This space is divided into two

  - **Old pointer space**: Contains survived objects that have pointers to other objects.
  - **Old data space**: Contains objects that just contain data(no pointer other objects). Strings, boxed numbers, and arrays of unboxed doubles are moved here after surviving in "New space" for two minor GC cycles.

- **Large object space**: This is where are larger that the size limits of other spaces live. Each object gets its own [mmap](https://en.wikipedia.org/wiki/Mmap)'d region of memory. Large objects are never moved by the garbage collector

- **Code-space**: This is where the **Just in Time(JIT)** compiler stores compiled code Blocks. This is the only space with executable memory(although `Codes` may be allocated in "Large object space", and those are executable, too)

- **Cell space, property cell space, and map space**: These spaces contain `Cells, PropertyCells` and `Maps`, respectively. Each of these spaces contains objects which are all the same size and has some constraints on what kind of objects they point to, which simplifies collection.

Each of these spaces is composed of a set of pages. A Page is a contiguous chunk of memory allocated form the operating system with `mmap` (or [MapViewOfFile](https://docs.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-mapviewoffile) on Windows). Each page is 1MB is size, except for Large object space.

## Stack

This is the stack memory area and there is one stack per V8 process. This is where static data including method/function frames, primitive values, and pointers to objects are stored. The stack memory limit can be set using the `--stack_size` V8 flag.

## V8 memory usage (Stack vs Heap)

Let's use the below JavaScript program

```JavaScript
class Employee {
  constructor(name, salary, sales) {
    this.name = name;
    this.salary = salary;
    this.sales = sales;
  }
}

const BONUS_PERCENTAGE = 10;

function getBonusPercentage(salary) {
  const percentage = (salary * BONUS_PERCENTAGE) / 100;
  return percentage;
}

function findEmployeeBonus(salary, noOfSales) {
  const bonusPercentage = getBonusPercentage(salary);
  const bonus = bonusPercentage * noOfSales;
  return bonus;
}

let john = new Employee("John", 5000, 5);
john.bonus = findEmployeeBonus(john.salary, john.sales);
console.log(john.bonus);
```

[this is demonstration of how does it works](https://speakerdeck.com/deepu105/v8-memory-usage-stack-and-heap)

- **Global scope** is kept in a "Global frame" on the Stack
- Every function call is added to the stack memory as frame-block
- All local variables including args and the return valus is saved within the function frame-block on the Stack
- All primitive tyoes are stored directly on the Stack. This applies to global scope as well
- All object types are created on the Heap and is referenced from the Stack using Stack pointers. Functions are just objects in JS. Also applies to global scope as well
- Functions called from the current function is pushed on top of the Stack
- When a function returns its frame is removed from the Stack
- Once the main process is complete, the objects on the Heap do not have any more pointers from Stack and becomes orphan
- Unless you make a copy exlicitly, all object references within other objects are done using reference pointers

Distinguished pointers and data on the heap is important for garbage collection and V8 uses the **"Tagged pointers"** approach to this -in this approach, it reserves a bit at the end of each word to indicate whether it is pointer or data. This approach requires limited compiler support, but it's simple to implement while being fairly efficient.

## V8 Memory management: Garbage collection

When a program tries to allocate more memory on the Heap that that is freely available(depending on the V8 flags set) we encounter **out of memory errors**. An incorrectlly managed heap could also a memory leak.

V8 manages the heap memory be garbage collection. In simple terms, it frees the memory used by orphan objects, i.e, objects that are no longer referenced from the Stack directly or inderectly (via a refernce in another object) to make space for new object creation.

> **Orinoco** is the codename of the V8 GC project to make use of parallel, incremental and concurrent techniques for garbage collection, to free the main thread.

The garbage collector in V8 is responsible for reclaiming the unused memory for reuse by the V8 process.

V8 garbage collector are generational(Objects in Heap are grouped by their age and cleared at dfferant sages), There are two stages and three different algorithms used for gabage collection by V8:

### Minor GC (Scavenger)

This type of GC keeps the young or new generation space compact and clean. Objects are allocated in new-space, which is fairly small (between 1 and 8 MB, depending on behavior heuristics). Allocation in "new space" is very cheap: there is an allocation pointer which we increment whenever we want to reserve space for a new object. When the allocation pointer reaches the end of the new space, a minor GC is triggered. This process is also called **Scavenger** and it implements [Cheney's algorithm](https://en.wikipedia.org/wiki/Cheney's_algorithm). It occurs frequently and uses parallel helper threads and is very fast.
[demonstration](https://speakerdeck.com/deepu105/v8-minor-gc)

1. Let us assume that there are already objects on the "from-space” when we start(Blocks 01 to 06 marked as used memory)
2. The process creates a new object(07)
3. V8 tries to get the required memory from from-space, but there is no free space in there to accommodate our object and hence V8 triggers minor GC
   4)Minor GC recursively traverses the object graph in "from-space" starting from stack pointers(GC roots) to find objects that are used or alive(Used memory). These objects are moved to a page in the "to-space". Any objects reference by these objects are also moved to this page in "to-space" and their pointers are updated. This is repeated until all the objects in "from-space" are scanned. By end of this, the "to-space" is automatically compacted reducing fragmentation
4. Minor GC now empties the "from-space" as any remaining object here is garbage
5. Minor GC swaps the "to-space" and "from-space", all the objects are now in "from-space" and the "to-space" is empty
6. The new object is allocated memory in the "from-space"
7. Let us assume that some time has passed and there are more objects on the "from-space" now(Blocks 07 to 09 marked as used memory)
8. The application creates a new object(10)
9. V8 tries to get required memory from "from-space", but there is no free space in there to accommodate our object and hence V8 triggers a second minor GC
10. The above process is repeated and any alive objects that survived the second minor GC is moved to the "Old space". First-time survivors are moved to the "to-space" and the remaining garbage is cleared from "from-space"
11. Minor GC swaps the "to-space" and "from-space", all the objects are now in "from-space" and the "to-space" is empty
12. The new object is allocated memory in the "from-space"
    So we saw how minor GC reclaims space from the young generation and keeps it compact. It is a stop-the-world process but it's so fast and efficient that it is negligible most of the time. Since this process doesn't scan objects in the "old space" for any reference in the "new space" it uses a register of all pointers from old space to new space. This is recorded to the store buffer by a process called [write barriers](https://www.memorymanagement.org/glossary/w.html#term-write-barrier).

### Major GC

This type of GC keeps the old generation space compact and clean. This is triggered when V8 decides there is not enough old space, based on a dynamically computed limit, as it gets filled up from minor GC cycles.

The Scavenger algorithm is perfect for small data size but is impractical for large heap, as the old space, as it has memory overhead and hence major GC is done using the **Mark-Sweep-Compact** algorithm. It uses a **tri-color**(white-grey-black) marking system. Hence major GC is a three-step process and the third step is executed depending on a fragmentation heuristic.

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

Let us look at the major GC process:

1) Let us assume that many minor GC cycles have passed and the old space is almost full and V8 decides to trigger a "Major GC"
2) Major GC recursively traverses the object graph starting from stack pointers to mark objects that are used as alive(Used memory) and remaining objects as garbage(Orphans) in the old space. This is done using multiple concurrent helper threads and each helper follows a pointer. This does not affect the main JS thread.
3) When concurrent marking is done or if the memory limit is reached the GC does a mark finalization step using the main thread. This introduces a small pause time.
4) Major GC now marks all orphan object's memory as free using concurrent sweep threads. Parallel compaction tasks are also triggered to move related blocks of memory to the same page to avoid fragmentation. Pointers are updated during these steps.
