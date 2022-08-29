# 🚀 Operating System: Process and Process Management

Process: In simple words, a process is an instance of an executing application. An application is a file containing a list of instructions stored in the disk (often called an executable file), in flash memory, maybe in the cloud but it’s not executing, it’s a static entity. When an application is launched it is loaded into the memory and it becomes a process, so it is an active entity with a program counter specifying the next instruction to execute and a set of associated resources. If the same program is launched more than once then multiple processes will be created executing the same program but will be having a very different state.

A process **encapsulates** all the data for running application, this includes the text, the code of the program, a data section, which contains global variables and **data** which are available when the process is first initialized. As text and the data are available when the process is first initialized they are called static states and are available when the process first loads.

The process also encapsulates the process **stack** which contains temporary data (such as function parameters, return addresses, and local variables), it is a dynamic part of the process state which grows and shrinks during execution in Last-in-First-out order. Suppose we are executing a function “A” and want to call another function “B” from this function, for this, we have to save our current state (state of function “A”) to the stack and jump to execute function “B”, after the execution of function “B”, the state of the previous function (“A”) from which the function “B” was called is restored from the top of the stack and function “A” can continue its execution from that vary instruction it had left (program counter was also saved).
![image](https://user-images.githubusercontent.com/49281851/179557934-6416675e-8de3-4650-bb97-09ba48cbf64d.png)

A process may also include a **heap**, which is the memory that is dynamically allocated during process run time. Text and data, stack and heap are the types of state of the process.

## Process Control Block (PCB)

> A Process Control Block is the data structure that operating system maintains for every single process. The PCB must contain process states like program counter, stack pointer, all the value of the CPU register, various memory mapping from virtual to physical memory, it may also include a list of open files, information which is necessary for scheduling of the process like how much time this particular process has executed on CPU in the past and how much time it should be allocated to execute in the future.

A PCB is created at the very moment a process is created with some initializations like PC points to the first instruction that needs to be executed. Certain field of the process changes as the state of the process changes for example whenever a process asks for more memory, the OS will allocate more memory to the process and update certain values of the PCB like Page table, virtual memory limits, etc. Some values of the process change too often like the value of program counter which changes on the execution of every single instruction. As updating such changes in PCB can be an expensive task, their values are stored and updated in CPU registers which are very fast. However, OS updates all the values maintained by CPU registers to the PCB whenever that particular process is no longer running on the CPU.

Suppose OS is managing two processes p1 and p2, currently, p1 is running and p2 is idle, their PCBs are stored somewhere in the memory. The process p1 is currently running means that CPU registers hold the values that correspond to the state of p1. After some time suppose OS decides to interrupt p1, so OS will update all the state information fields of the process p1 including the CPU registers to the PCB of p1. After this the OS will restore the PCB of p2 from the memory, i.e. OS will update all the CPU register from the PCB of p2 and will start executing process p2. After some time if process p2 is interrupted the PCB of p1 will be restored and CPU registers will be updated from the value of PCB of p1 and p1 will start executing from the exact same point where it was interrupted earlier by the operating system. Each time this swapping is performed we call it context switch.

## Context Switch

It is a mechanism used by operating system to switch execution from the context of one process to the context of another process.

Context switching is an expensive operation because of the number of instructions involved in loading and restoring values of fields of PCB from the memory. Also when a process p1 is executing a lot of its data is stored in the CPU cache as accessing the cache is much much faster as compared to accessing from the memory. When the data we want is present in the cache we say that cache is hot. When CPU will switch the context from process p1 to process p2, the process p2 will replace process p1’s cache with its own cache, so next time when the context will switch from the process p2 back to the process p1, p1 will not find its data in the cache and has to access it from the memory, so it will incur cache misses, so we call it cold cache.

## Process Creation

In the operating system, a process can create a child process. Hence all processes come from a single root in which a creating process is the parent process and the created process is the child of that process. Once the initial boot process is done the and operating system is loaded, it will create some initial process. When user logs into the system a user shell process are created, and when the user types in the command (emacs, nano, etc) then new process get spawned from that shell parent process. So the final relationship looks like a tree.

## The mechanism for process creation

Most operating systems support two basic mechanisms for process creation
![image](https://user-images.githubusercontent.com/49281851/179558016-a08dc53a-c73c-44fe-9b45-be5c619df1c8.png)

Fork: With this mechanism, the operating system will create a new child process with PCB and then it copies all the values of parent’s PCB to child’s PCB. After that, both the child and the parent continues their execution at instruction just after the fork call because both processes contain exact same values in their PCB which also includes program counter.
Exec: This replaces the child’s image and loads the new program. Child’s PCB contains the new initialized value and program executes from the beginning.
The mechanism of creating a new program is like calling the fork which creates a child process with exact same PCB as that of the parent and then calling exec which replaces the child’s image with the new program’s image.

## CPU Scheduling

At a time there can be multiple processes waiting in the ready queue. The CPU scheduler determines which of the currently running process should be dispatched to the CPU for execution, and how long it should take.

In order to manage the CPU, the operating system must be able to preempt i.e. to interrupt the current running process and save is current context. This operation is called preemption. Then operating system must run the scheduling algorithm in order to choose the next process to run. And at last, once the process is chosen the operating system must dispatch this process to the CPU and switch to its context. OS must make sure that CPU is spending more time on running processes and not executing scheduling algorithm, dispatching, preempting or doing some other OS operations. Hence it is important to have an efficient design and as well as efficient implementation of the various algorithms involved for example scheduling. Also, efficient data structures that are required to represent waiting processes in the ready queue or any other information (like the priority of the processes, how long the algorithm ran in the past) that are relevant to make scheduling decisions.
![image](https://user-images.githubusercontent.com/49281851/179558064-0308217c-33d0-41cc-a81a-7dbf05dac94a.png)

## Inter-process communication

Many applications are structured as multiple processes, so these multi processes have to able to interact with each other in order to achieve a common goal. As we have already studied that operating system isolates the process from each other in order to protect each other’s memory space, OS controls the amount of CPU each application gets. So some communication mechanism is required to build without sacrificing the protection. These mechanisms are called inter-process communication (IPC). Their task is to transfer data/info between address spaces without sacrificing the protection and isolation that OS provides. As communication can vary from the continuous stream of data sharing, periodic data sharing, a single piece of data sharing, etc so the IPC mechanism has to be flexible with good performance.

- **Message-Passing IPC**: In this mechanism, the operating system establishes a communication channel (like shared buffer), and processes interact with each other by writing or sending data to the channel and reading or receiving data from the channel.
  - **Advantage**: Advantage of this mechanism is that OS manages both writing and reading data from the channel and provides APIs. So both process uses the exact same APIs.
  - **Disadvantage**: One disadvantage of this mechanism is that data has to first copy from sending process memory space to shared channel and then back to receiving process memory space.
- **Shared Memory IPC**: In this mechanism, the OS creates a shared memory channel and then maps it to each process memory space, and then processes are allowed to read and write to the channel as if they would do to any memory space that is part of their memory space.
  - **Advantage**: The advantage of this process is that OS is not involved in the communication.
  - **Disadvantage**: As OS is not involved in the communication this mechanism does not support fixed and well-defined APIs for reading and writing data, so this mechanism is error-prone and sometimes the developers have to re-implement the code.
