# Operating System - I/O Hardware

One of the important jobs of an OS is to manage various I/O devices including mouse, kbd, dis drives, display adapters, USB devices. Bit-mapped screen, LED, Analog-to-digit converter, On/off, network connections, audio I/O, printers etc.

An I/O system is required to take an application I/O request and send it to the physical device, then take whatever response comes back th device and send it to the application. I/O devices can be divided into two categories:

- **Block devices** - A block device is one with which the driver communicates by sending entire blocks of data. For example, Hard disks, USB cameras, Disk-On-Key etc.
- **Character devices** - A character device is one with which the driver communicates by sending and receiving single characters(bytes, octets). For example, serial ports, parallel portss, sounds cards and etc

## Device Controllers

Device drivers are software modules that can be plugged into an OS to handle a particular device. Operating System takes help from device drivers to handle all I/O devices.

The Device Controller works like an interface between a divice and a device driver. I/O units (Keyboard, mouse, printer, etc ) typically consist of a mechanical component and an electronic component where electronicc component is called the device controller.

There is always a device controller and a device driver for each device to communicate with the OSs. A device controller may be able to handle multiple devices. As an interface its main task is convert serial bit stream to block of bytes, perform error correction as mecessary.

Any device connected to the computer is connected by a plug and socket, and the socket is connected to a device controller. Following is a model for connecting the CPU, memory, controller, and I/O devices where CPU and device controllers all use a common bus for communication

## Synchronous vs Asynchronous I/O

- **Synchronous I/O** - In this scheme CPU execution waits while I/O proceeds
- **Asynchronous I/O** - I/O proceeds concurrently with CPU execution

## Comminication to I/O Devices

The CPU must have a way to pass info to and from an I/O device. There are three approaches available to communicate with th CPU and Device.

- Special Instruction I/O
- Memory-mapped I/O
- Direct memory access (DMA)

## Special Instruction I/O

This uses CPU instruction that are made for contorlling I/O devices. These instructions typically allow data to be sent to an I/O device or read from an I/O device

## Memory-mapped I/O

When using memory-mapped I/O, the same address space is shared by memory and I/O devices. The device is connected directly to certain main memory locations so that I/O device can transfer block of data to/from memory without going through CPU.

![image](https://user-images.githubusercontent.com/49281851/190649502-81a2bef8-1ff1-45e3-a203-545c4afbf0dc.png)

While using memory mapped I/O, OS allocates buffer in memory and informs I/O device to use that buffer to send data to the CPU. I/O device operates asynchronously with CPU, interrupts CPU when finished.

The advantage to this method is that every instruction which can access memory can be used to manipulate an I/O device. Memory mapped IO is used for most high-speed I/O devices like disks, communication interface

## Direct Memory Access (DMA)

DMA means CPU grants I/O module authority to read from or write to memory without involvement. DMA module constrols exchange of data between main memory and I/O device. CPU is only involved at the beginning and end of the transfer and intrrupted only after entire block has been transferred.

DMA needs hardware DMA controller **DMAC** that manages the data transfer and arbitrates acces to the system bus. The controllers are progreammed with source and destination pointers (where to read/write), counters to track the number of transferred bytes, and settings, which includes I/O and memory types, interrupts and states the CPU cycles.

![image](https://user-images.githubusercontent.com/49281851/190649558-edeb7cd9-5b94-45e6-a505-34634a228d5c.png)

1. Device driver is instructed to transfer disk data to a buffer address X
2. Device driver then instruct disk controller to transfer data to buffer
3. Disk controller starts DMA transfer
4. Disk controller sends each byte to DMA controller
5. DMA controller transfers bytes to bufferm the increases the memory address, decreases the counter C until C becomes zero
6. When C becomes zero, DMA interrupts CPU to signal transfer completion

## Polling vs Interrupts I/O

A computer must have a way of detecting the arrival of any type of input. There are two ways that this can happen, known as **polling** and **interrupts**. Both of these techniques allow the processor to deal with events that can happen at any time and that are not related to the process it is currently running.

### Polling I/O

Polling is the simplest way for an I/O device to communicate with the processor. The process of periodically checking status of the device to see if it is time for the next I/O operation, is called polling. The I/O device simply puts the information in a Status register, and the processor must come and get the information.

### Interrupts I/O

An alternative scheme for dealing with I/O is the interrupt-driven method. An interrupt is a signal to the microprocessor from a device that requires attention.

A device controller puts an interrupt signal on the bus when it needs CPU’s attention when CPU receives an interrupt, It saves its current state and invokes the appropriate interrupt handler using the interrupt vector (addresses of OS routines to handle various events). When the interrupting device has been dealt with, the CPU continues with its original task as if it had never been interrupted.
