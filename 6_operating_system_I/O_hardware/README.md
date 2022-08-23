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
The CPU  must have a way to pass info to and from an I/O device. There are three approaches available to communicate with th CPU and Device.
- Special Instruction I/O
- Memory-mapped I/O
- Direct memory access (DMA)

## Special Instruction I/O
This uses CPU instruction that are made for contorlling I/O devices. These instructions typically allow data to be sent to an I/O device or read from an I/O device

## Memory-mapped I/O
When using memory-mapped