# 🚀 Operating System: Process and Process Management

Application: Instructions (executable file).
Process: When application runs, it becomes the process. If the same program is launched more than once then multiple processes will be created executing the same program but will have diff state.

A process **encapsulate** all the data for running application (text, the code of program and **data** that is available when the the process first initialiazed). After all data is available the process will call this data.

The process also encapsulates the process **stack** which contains temporary data (such as function parameters, return addresses, and local variables), it is a dynamic part of the process state which grows and shrinks during execution in Last-in-First-out order.
![image](https://user-images.githubusercontent.com/49281851/179557934-6416675e-8de3-4650-bb97-09ba48cbf64d.png)

A process may also include a **heap**, which is the memory that is dynamically allocated during process

## Process Control Block (PCB)

> A PCB is the data structure that OS maintains for every single process. The PCB must contain process states like program counter, stack pointer, all the value of the CPU register, various memory mapping from virtual to physical memory, it may also include a list of open files, information which is necessary for scheduling of the process like how much time this particular process has executed on CPU in the past and how much time it should be allocated to execute in the future.

A PCB is created at the very memont a process, with initializations like PC points to the first instruction that needs to be executed. The OS will allocate memory to the process and update certain values of the PCB. As updating such changes in PCB can be an expensive task, their values are stored and updated in CPU registers which are very fast. However, OS updates all the values maintained by CPU registers to the PCB whenever that particular process is no longer running on the CPU.

## Context Switch

Itsa mechanism used by OS to switch execution from the context of one process to the another.

Context switching is an expensive operation because of the number of instructions involved in loading and restoring values of fields of PCB from the memory. Also when a process p1 is executing a lot of its data is stored in the CPU cache as accessing the cache is much much faster as compared to accessing from the memory. When CPU will switch the context from process p1 to process p2, the process p2 will replace process p1’s cache with its own cache

## Process Creation

In the operating system, a process can create a child process. Hence all processes come from a single root in which a creating process is the parent process and the created process is the child of that process

## The mechanism for process creation

Most operating systems support two basic mechanisms for process creation
![image](https://user-images.githubusercontent.com/49281851/179558016-a08dc53a-c73c-44fe-9b45-be5c619df1c8.png)

Fork: With this mechanism, the operating system will create a new child process with PCB and then it copies all the values of parent’s PCB to child’s PCB. After that, both the child and the parent continues their execution at instruction just after the fork call because both processes contain exact same values in their PCB which also includes program counter.

## CPU Scheduling

At a time there can be multiple processes waiting in the ready queue. The CPU scheduler determines which of the currently running process should be dispatched to the CPU for execution, and how long it should take.
![image](https://user-images.githubusercontent.com/49281851/179558064-0308217c-33d0-41cc-a81a-7dbf05dac94a.png)

## Inter-process communication

So some communication mechanism is required to build without sacrificing the protection. These mechanisms are called inter-process communication (IPC). Their task is to transfer data/info between address spaces without sacrificing the protection and isolation that OS provides

As communication can vary from the continuous stream of data sharing, periodic data sharing, a single piece of data sharing, etc so the IPC mechanism has to be flexible with good performance.

- **Message-Passing IPC**: In this mechanism, the operating system establishes a communication channel (like shared buffer), and processes interact with each other by writing or sending data to the channel and reading or receiving data from the channel.

- **Shared Memory IPC**: In this mechanism, the OS creates a shared memory channel and then maps it to each process memory space, and then processes are allowed to read and write to the channel as if they would do to any memory space that is part of their memory space.
