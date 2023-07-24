# What’s the Diff: Programs, Processes, and Threads

## Programs
A program is the code tha t is stored on your computer that is intended to fulfill a certain task. There are many types of programs, including programs that help your computer function and are part of the operating system, and other programs that fulfill a particular job. These task-specific programs are also known as **application** and can incllude programs such as word processing, web browsing, and so on.

Programs are sotored on disk or in non-volatile memory in a form that can be executed by your computer. Prior to that, they are created using a programming lang as Lisp, Pascal. The end result is a text file of code that is compiled into binary from (ones and zeros) in order to run on the computer. Another type of program is called **interpreted** and instead of being compiled in advance in order to run, is interpreted into executable code at the time it run. Some common, typically interpreted programming languages are Python, PHP, JS and Ruby.

The end result is the same, however, in that when a program is run, it is loaded into memory in binary form. The computer's central processing unit **CPU** understands only binary instructions, so that's the form the program needs to be in when it runs. 

## How process works 

The program has been loaded into the computer's memory in binary form.

An executing program needs more than just binary code that tells the computer what to do. The program needs memory and various operating system resources in order to run. A **process** is what we call a program that has been loaded into memory along with all the resources it needs to operate.The **“operating system”** is the brains behind allocating all these resources

# How concurency works and difference between parallelism
Concurrency and parallelism are related concepts in computer science that deal with the execution of multiple tasks or processes. While they share some similarities, they are distinct concepts that refer to different ways of handling concurrent operations.

### Concurrency
Concurrency refers to the ability of a system to execute multiple tasks or processes in overlapping time periods. It allows different tasks to make progress simultaneously, even if they dont execute are the exact same time. Concurrency is commonly used in systems where tasks can be partially completed independently or when there is a need to respond to multiple events or users concurrently.

In concurrent systems, a single processor (or core) may switch between executing different tasks or processes, giving the illusion of parallel execution. However, its essentioal to note that in concurrent system, the tasks do not necessarily run simultaneously. They may execute for short intervals in an interleaved manner.

Concurrenct is often achieved throug techniques like multitasking, multithreading, and asynchronous programming. For example, in a multitasking operating system, the CPU time is shared between different processes, allowing them to make progress concurrently

### Parallelism
Parallelism, on the other hand, involves the simultaneous execution of multiple tasks or processes to achieve higher performance and speed up the overall execution time. It requires multiple processing units (CPU cores) that can independently processed simultaneously

### Key differences: 
1) **Nature of Execution**: Concurrency allows tasks to make progress in overlapping time periods, often through interleaved execution on a single processing unit. Parallelism involves simultaneous execution of tasks on separate processing units.
2) **Processor Requirement**: Concurrency can be achieved with a single processor, while parallelism requires several
3) **Task independance**: In concurrency, tasks may interact and share resources, while parallelism requires tasks to be largly independent to be executed simultaneously
