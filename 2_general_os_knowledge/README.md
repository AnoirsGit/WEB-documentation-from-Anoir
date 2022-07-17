# [How Operation System work. Basics](https://infinite.education/view/how_oss_work_in_general) 

The computer processor knows how to execute commands. But it needs someone to tell it which command to execute. This is the job of the operating system (OS). Every time you turn on your computer, it reads the first bytes from the disk drive, where it expects to see the commands that will start the operating system.

## What is Operating System?

An OS is a program for controlling hardware and running other programs. In OS terminology, a running program is called a process, and the OS is responsible for making sure that processes do not interfere with each other, even if they are running on the same system at the same time. It is also the responsibility of the OS to provide hardware for processes, for example, the OS retains the old direct control over hardware devices, and processes interact with devices only through system calls provided by the OS. The OS also provides a file system that abstracts from storage devices. Finally, the OS provides a user interface for running programs and managing the file system.

## Device driver
**OS plug-in module for controlling particular device**
![image](https://user-images.githubusercontent.com/49281851/179426361-77273e5a-923c-412a-8756-c8df719c51ed.png)


A **device driver** is an OS plug-in that controls a specific I/O device. Some standard devices can work with a generic driver, such as a USB mouse. Many devices require a more specific driver for certain hardware devices, such as the GPU.

The main goal of a modern operating system is to allow multiple processes to run simultaneously (at the same time). The problem is that each processor core can only execute code for one process at a time, and the OS' own code cannot execute on a core at the same time as any process.

The solution is to have each processor core switch between running each open process and alternate running processes with running OS code.
![image](https://user-images.githubusercontent.com/49281851/179426395-02f8b6df-03cb-4abe-a617-333390ce6e0d.png)

> **Example:** we have two processor cores and three open processes **A, B** and **C**. Note that each process only runs on one core at a time, at no point in time is process B running on two cores at the same time, also note that the OS code always runs on each core between each process. Here is the part of the OS called the scheduler, which runs after each process, to decide which OS job, if any, should be done and which process should run next.
The question is how the current process is interrupted. If a running process is left alone, it will continue indefinitely. However, when a hardware interrupt occurs, the interrupt handler passes control to the scheduler, rather than the processor core returning to the interrupted process. The scheduler then decides which OS code to run, if any, and which process should run next.

## Pre-empitive multitasking
1) CPU recieves interrupt
2) interrupt stores program
3) interrupt invokes handler
4) handler saves rest of state of the CPU for the process
5) handler does its buisiness
6) handler invokes the scheduler
7) scheduler selects a process to run
8) scheduler restores state of CPU for that process
9) scheduler jumps execution to that process

![image](https://user-images.githubusercontent.com/49281851/179426423-095947dc-92a6-4a0c-ab84-a06f96482798.png)

The question is in two things, **1** and **7**. 

> First, what if an interrupt is not initiated by any device for a long time. This will allow the current process to take over the CPU, whereas normally we want each process to at least get some time on a regular basis, for example every few seconds, but video games run frame by frame, not by second, and it would be no good if some other process ran uninterrupted for a second or more. To ensure that the scheduler runs regularly, whether or not the I/O devices need attention, the sync device on the motherboard is configured to send an interrupt every 10-20 milliseconds on a regular basis. In this way, the system ensures that the scheduler gets a chance to change the running process on each core at least several times. The next question is how the scheduler chooses which process to run next. Using the simplest algorithm **round-robin**, the scheduler simply starts each process in turn, which guarantees that each process is started regularly. The more complex algorithms used in modern operating systems try to take into account which cross-robin processes take more time than the others.  
![image](https://user-images.githubusercontent.com/49281851/179426452-3da0e668-3802-42aa-ba38-0973f53b8021.png)

> - Not only do the processes share the processor cores, they must also share the system memory. The task of the OS is to regulate the use of memory by the processes, so that each process does not interfere with the parts of memory used by other processes and the OS itself.
> - The OS is in charge of the system, each process can only access its own part of memory (*a hardware-imposed restriction*). We need a loophole for this restriction, because processes should be able to call basic procedures at fixed addresses in a part of the OS memory. These procedures, called **system calls**, are the means by which processes initiate requests to the operating system.
> System calls provide functionality, for example to read and write files or to send and receive data over a network. To call a system call the process must use a special processor instruction, usually called **CISCO**, in which the process specifies the system call number. When this instruction is called, the processor looks up the procedure address corresponding to the number in the system call table and moves to that execution address. Since the OS controls the system call table, the process can only move to execution at the addresses chosen by the OS.
> - Aside from loophole, how do the OS and hardware restrict the process to only access its own portion of memory:   
        1) The CPU operates at two different privilege levels. When OS code is executed, the CPU goes to a privilege level that allows access to I/O devices and any memory address. When a process is running, the processor goes to the privilege level, which causes a hardware exception when the code attempts to directly access I/O devices or addresses not allowed for that process.
        2) Each process uses part of its memory for the stack, the heap and for storing the process code in a section which is confusingly called (text section), although the code is in binary form. The code section is simple: the binary instructions are stored in a continuous chunk of memory and never change throughout the runtime, except when dunamic linking with shared libraries. The stack and the heap store data, the difference being that the stack stores local variables used by the process and the heap stores everything else. Many processors, including the x86, have a special register for storing these addresses, usually called (the stack pointer). The size of some systems is tracked by another pointer, usually called the stack boundary, which is stored in another processor register. In PC it is customary to store the stack at the top of the process address space and the process code text at the bottom, all the remaining space in between is available to the heap.
        3) The process memory does not actually refer directly to the actual bytes of the system memory, instead fragments of the process address space are mapped by the operating system to fragments of the system memory, but necessarily contiguous and in the same order. Each process has its own full address space, and the OS keeps a separate memory table for each process. Efficiently the OS can locate a process in any part of RAM and each process can access only its own memory area. The memory areas in which processes store their data are called (pages). Each page usually has a certain size which depends on the processor. 32-bit x86 processors, for example, typically use pages of 4 kB. To free up valuable RAM, the OS may decide to push the process pages into storage, usually on the hard disk.
        ![image](https://user-images.githubusercontent.com/49281851/179426485-3d5f45da-3a39-4ca1-b242-dbc754ed96cb.png)

> -  **CREATED -> WAITING <-> RUNNING -> TERMINATED**   
![image](https://user-images.githubusercontent.com/49281851/179426502-40732383-f5fd-4d53-9cbe-25003f714f7a.png)

>        In this lifecycle, the process goes through several basic states. After the OS has done all the things it needed to do at the time the process was created, the process goes into a **waiting state**. The point of waiting here is to wait for the scheduler to choose, when the scheduler chooses a process to run, it of course goes into an execution state, then the scheduler chooses another process to run on the same core, that process goes into a waiting state again, and the process goes back and forth many times between waiting and execution until the process is finished, at which point it goes into its final state (completed). There is another important state, and that is **BLOCKED**, in the locked state the process is waiting for some external event in the system before it can continue, rather than waiting to be scheduled. Thus, it is neither in a state of execution nor in a so-called wait state. More often than not, a blocked state occurs when a process invokes system calls, such as file reads. Reading a file often locks the process because most storage devices, such as a hard disk drive, are relatively slower than CPU operations, and the program cannot do anything useful until it reads the file, In such cases the process can free the CPU core it was using and take itself out of the waiting pool, allowing other processes to work while waiting, and when the OS finishes receiving the requested data, it will unlock the process and put it back into the waiting state for the scheduler to re-partition The scheduler only selects processes in the standby state until the process returns to the standby state again.
