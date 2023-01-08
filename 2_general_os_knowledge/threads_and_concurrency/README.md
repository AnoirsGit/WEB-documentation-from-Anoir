# What’s the Diff: Programs, Processes, and Threads

## Programs
A program is the code tha t is stored on your computer that is intended to fulfill a certain task. There are many types of programs, including programs that help your computer function and are part of the operating system, and other programs that fulfill a particular job. These task-specific programs are also known as **application** and can incllude programs such as word processing, web browsing, and so on.

Programs are sotored on disk or in non-volatile memory in a form that can be executed by your computer. Prior to that, they are created using a programming lang as Lisp, Pascal. The end result is a text file of code that is compiled into binary from (ones and zeros) in order to run on the computer. Another type of program is called **interpreted** and instead of being compiled in advance in order to run, is interpreted into executable code at the time it run. Some common, typically interpreted programming languages are Python, PHP, JS and Ruby.

The end result is the same, however, in that when a program is run, it is loaded into memory in binary form. The computer's central processing unit **CPU** understands only binary instructions, so that's the form the program needs to be in when it runs. 

## How process works 

The program has been loaded into the computer's memory in binary form.

An executing program needs more than just binary code that tells the computer what to do. The program needs memory and various operating system resources in order to run. A **process** is what we call a program that has been loaded into memory along with all the resources it needs to operate.The **“operating system”** is the brains behind allocating all these resources