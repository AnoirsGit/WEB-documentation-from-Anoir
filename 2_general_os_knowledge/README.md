# [How Operation System work. Basics](https://infinite.education/view/how_oss_work_in_general) 

Computer CPU knows how to execute commands. But it needs someone to tell him what command to execute. It is the job of Operation System (OS). Whenever you power your computer, it reads first bytes on your Disk Drive, where it expects to see commands that will start Operation System. 

## What is Operating System?

OS is a progrma form naging the hardware and running other programs. In OS terminology a runnig program is a called a process and it is the OS responsibility to keep the processes from interfering with one another even if they run on the same system at the moment. Also in OS responsibility to provide hardware for the processes such as OS retains old direct control of the hardware devices with the processes only interacting with the devices called system calls provided by the OS. OS also provides a file system which abstract over the storage devices. Lastly OS provides a user interface for user to run programs and manage file system

## Device driver
**OS plug-in module for controlling particular device**

A **device driver** is a plug-in module of the OS that handles the management of a particular I/O device. Some standardized devices may function with a generic driber as USB mouse. Many devices require more specific driver for some hardware devices as GPU.

A primary purpsoe of a modern OS is to allow for multiple processes to run concurently(at the same time). A problem is that each CPU core can only execute the code of one process at a time and the OS own code can't run on a core at the same time as any process. 

The solution is to have each CPU core to laternate between running each open process and alternate running processes with running OS code.
> **Example:** For example we have a two CPU cores and three open processes a *A,B* and *C* notice that each process only runs one core at a time at no point deos say process B run simultaneously on ot the two cores, also notice that OS code always runs on each core in between each process. Here is that portion  of the OS called the scheduler runs after each process to decide what OS work if any should be done and which process should run next.   
The question is how does the currently running process get interrupted. Left on its own a running process would continue indefinetely. When any hardware interrupt is triggered however the interrupt handler passes off controll to the scheduler rather than handling the processor core back to the interrupted process. The scheduler then decides what OS code to run if any and what process should run next 

3:37