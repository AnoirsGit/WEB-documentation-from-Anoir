# 🚀 Visualizing memory management in Golang

Go is as statically typed & compiled language like C/C++ and Rust. Hence Go does not need VM and Go aplication binaries include a small runtime embedded in them to take care of language features like Garbage collection, scheduling & concurrency.

## Go internal memory structure

> Go runtime schedules Goroutines(`G`) onto Logical Process (`P`) for execution. Each `P` has machine (`M`). We will use `P`, `M` & `G` throughout this post. [Go scheduler](https://www.ardanlabs.com/blog/2018/08/scheduling-in-go-part2.html)

![image](https://user-images.githubusercontent.com/49281851/184501890-7fa5ee6b-1cd2-4188-9c75-9b78962c1ac9.png)

Each Go program processes is allocated some viral memory by the OS, this is the total memory that the process has access to. The actual memory that is used within the virtual memory is called **Resident Set**. This space is managed by the internal memory constructs as below:

![image](https://user-images.githubusercontent.com/49281851/184501957-30d5dfb3-e75a-4043-9a48-04b59d1da70d.png)

This is a view based on the internal object used by Go, In reality, Go divides and groups memory into pages as described [here](https://blog.learngoprogramming.com/a-visual-guide-to-golang-memory-allocator-from-ground-up-e132258453ed)

## Page Heap(`mheap`)

This is where Go stores dynamic data(any data which size cannot be calculated at compile time). This is the biggest block of memory and this is where **Garbage Collector** takes place.

The resident set is divided into pages of `8KB` each and is managed by one global `mheap` object.

> Large objects(Object of Size > 32kb) are allocated directly from `mheap`. These large requests come at an expense of central lock, so only one `P`’s request can be served at any given point in time.

`mheap` manages pages grouped into different constructs as below:

- **`mspan`**: `mspan` is the most basic structure that manages the pagesof memory in `mheap`. It's a double-linked list that holds the address of the start page, span size class, and the number of pages in the span. Like TCMalloc, Go also divides Memory Pages into a block of 67 different classes by size starting at 8bytes up to 32Kbytes as in the below image: ![image](https://user-images.githubusercontent.com/49281851/184501986-6291e254-0e7d-4568-a2e5-dd0b802ab095.png) Each span exists twice, one for objects with pointers (**scan** classes) and one for objects with no pointers (`nosacan` classes). This helps during GC as `noscan` spans need not to be traversed too look for live objects.

- **`mcentral`**: `mcentral` groups spans of smae size class together. Each `mcentral` contains two `mspanList`:
  - **empty**: Double linked list of spans with no free objects or spans that are cached in a `mcache`. When span here is feeed, its moved to the nonempty list
  - **nonempty**: Double linked list of spans with a free object. When a new span is requested from `mcentral`, it takes that from the nonempty list and moves it into the empty list

When `mcentral` doesn't have any free span, it requests a new run of pages from `mheap`.

- **`arena`**: The heap memory grows and shrinks as required within the virtual memory allocated. When more memory is needed, `mheap` pulls them from the virtual memory as a chunk of 64MB(for 64-bit architecture) called `arena`. The pages are mapped to spans here.

- **`mcache`**: This is a very interesting construct. `mcache` is a cache of memeory provided to a `P`(Logical Processor) to store small objects(Object size <=32Kb). Though this resembles the thread stack, it is part of the heap and is used for dynamic data. `mcache` contains `scan` and `nonscan` types of `mspan` for all class sizes. Goroutines can obtain memory from `mcache` without any locks as a `P` can have only one `G` at a time. Hence this is more efficient. `mchache` requests new spans from `mcentral` when required

## Stack

This is the stack memory area and there is one stack per Goroutine(`G`). This is where static data including function frames, static structs, primitive values, and pointers to dynamic structs are stored. This is not the same as `mcache` which is assigned to a `P`

## Go memory usage (Stack vs Heap)

```Golang
package main

import "fmt"

type Employee struct {
  name   string
  salary int
  sales  int
  bonus  int
}

const BONUS_PERCENTAGE = 10

func getBonusPercentage(salary int) int {
  percentage := (salary * BONUS_PERCENTAGE) / 100
  return percentage
}

func findEmployeeBonus(salary, noOfSales int) int {
  bonusPercentage := getBonusPercentage(salary)
  bonus := bonusPercentage * noOfSales
  return bonus
}

func main() {
  var john = Employee{"John", 5000, 5, 0}
  john.bonus = findEmployeeBonus(john.salary, john.sales)
  fmt.Println(john.bonus)
}
```

One major difference Go has compared to many garbage collected languages is that many obnjects are allocated directly on the program stack. The Go compiler uses a process called [escape analysis](https://www.ardanlabs.com/blog/2017/05/language-mechanics-on-escape-analysis.html) to find objects whose lifetime is known at compile-time and allocates them on the stack rather than in garbage-collected heap memory.
During compilattion, Go does the esvape analysisi to determine what can go into Stack and what need to go into Heap. We can see these details during compilation by running `go build` with `-gcflags '-m'` flag.

[visualization](https://speakerdeck.com/deepu105/golang-memory-usage-stack-vs-heap)

- **Main** function is kept in a "main frame" on the Stack
- Every func call is added to the stack memory as a frame-block
- All static variables including args and the return value is saved within the function frame-block on the Stack
- All static values regardless of type are stored directly on the Stack. This applies to global scope as well
- All dynamic types are created on the Heap and is referenced from the Stack uning Stack pointers. Objects of size less than 32Kb go to the `mcache` of the `P`. This applies to global scope as well
- The struct with static data is kept on the stack until any dynamic value is added at that point the struct is moved to the heap
- Functions called from the current func is pushed on top of the Stack
- When a function return its frame is removed from the Stack
- Once the process is complete, the objects on the Heap do not have any more pointers from Stack and becomes orphan


The Stack as you can see is automatically managed and is done so by the operating system rather than Go itself. Hence we do not have to worry much about the Stack. The Heap, on the other hand, is not automatically managed by the OS and since it's the biggest memory space and holds dynamic data, it could grow exponentially causing our program to run out of memory over time. It also becomes fragmented over time slowing down applications. This is where garbage collection comes in.

## Go memory management 
Go's memory management involves automatic allocation when memory is needed and garbage collection when memory is not needed anymore. It's done by the standard library.

## Memory Allocation
Many programming languages that employ Garbage collection use a generational memory structure to make collection efficient along with compaction to reduce fragmentation. Go takes a different approach here, as we saw earlier, Go, structures memory quite differently. Go employs a thread-local cache to speed up small object allocations and maintains `scan`/`noscan` spans to speed up GC. This structure along with the process avoids fragmentation to a great extent making compact unnecessary during GC. Let's see how this allocation takes place.

Go decides the allocation process of an objec based on its size and is divided into three categories:

**Tiny(size < 16B)**: Objects of size less than 16 bytes are allocated using the mcache's tiny allocator. This is efficient and multiple tiny allocations are done on a single 16-byte block.
![image](https://res.cloudinary.com/practicaldev/image/fetch/s--iILSHo5v--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://i.imgur.com/Kh26oVp.gif)

**Small(size 16B ~ 32KB)**: Objects of size between 16 bytes and 32 Kilobytes are allocated on the corresponding size class(mspan) on mcache of the P where the G is running.
![image](https://res.cloudinary.com/practicaldev/image/fetch/s--FZK3_38e--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://i.imgur.com/PY4pZhq.gif)

In both tiny and small allocation if the `mspan`s list is empty the allocator will obtain a run of pages from the `mheap` to use `mspan`. If the `mheap` is emptyh or has no page runs large enough then it allocates a new group of pages (at least 1MB) from the OS.

**Large(size > 32KB)**: Objects of size greater than 32 kilobytes are allocated directly on te corresponding size class of `mheap`. If the `mheap` id empty or has no page runs large enough then it allocates a new group of pages (at least 1MB)from the OS.
![image](https://res.cloudinary.com/practicaldev/image/fetch/s--B2WWlqJJ--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://i.imgur.com/uLhLZMm.gif)

## Garbage Collection 
When a program tries to allocate more memory on the Heap than that is freely available we encounter ***out of memory errors***. An incorrectly managed heap could also cause a memory leak.

Go manages the heap memory by garbage collection. In simple terms, it frees the memory used by orphan objects, i.e, objects that are no longer referenced from the Stack directly or indirectly to make space for new object creation.

The process starts when a certain percentage(GC Percentage) of heap allocation are done and the collector does different phase of work: 

- **Mark Setup** (Stop the world): Whem GC starts, the collector turns on [write barries](https://www.memorymanagement.org/glossary/w.html#term-write-barrier) so data intgrity can be maintained during the next concurrent phase. This step needs a very small pause as every running Gorountine is paused to enable this and then continues.
- **Marking** (Concurrent): Once write barries are turned on the actual marking process is started in parallel to the application using 25% of the available CPU capacity. The corresponding `P`'s are reserved untill marking is complete. This is sone using dedicated Goroutines. Here the GC marks values in the heap that is alive (referenced from the Stack on any active Goroutines). When collection takes longer the process may employ active Gorountine from application to assist in the marking process. this is called ***Marking Assist***
- **Mark Termination** (Stop the world): Once marking is done every active Gorountine is paused and write barries are turned off and clean up tasks are started. The GC  also calculates the GC goal here. Once this is done the reserved `P` are released back to the application.
- **Sweeping** (Concurrent): Once the collection is done and allocations are attempted, the sweeping process starts to reclaim memory from the heap that is no markable alive. The amount of memory swept of synchronous to the amount being allocated.

[visualization](https://speakerdeck.com/deepu105/go-gc-visualized)

1) We are looking at a single Goroutine, the actual process does this for all active Goroutines. The write barriers are turned on first.
2) The marking process picks a GC root and colors it black and traverses pointers from it in a depth-first tree-like manner, it marks each object encountered grey
3) When it reaches an object in a noscan span or when an object has no more pointers it finishes for the root and picks up the next GC root object
4) Once all GC roots are scanned, it picks up a grey object and continues to traverse its pointers in a similar fashion
5) If there are any pointer changes to an object when write barriers are on, the object gets colored grey so that GC re-scans it
6) When there are no more grey objects left, the marking process is complete and the write barrier is turned off
7) Sweeping will take place when allocations start

This has some stop-the-world process but it's generally very fast that it is negligible most of the time. The coloring of objects takes place in the `gcmarkBits` attribute on the span.
