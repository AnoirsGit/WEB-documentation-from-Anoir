# Inter Process Communication (IPC)

A process can be of two types:

- Independent process.
- Co-operating porcess.

An independent pocess is not affected by the execution of other processes while a co-operating process can be affected by other executing processes. Though one can think that those poecesses, which are running independently, will wxwcute very efficiently, in reality, there are many situations when co-operative nature can be utilized for increasing computational speed, convenience, and modularity. Inter-process communication (IPC) is a mechanism that allows process to communicate with each other and synchronize their actions. The communication between these processes can be seen as a method of co-operation between them. Processes can communicate with each other though both:

![image](https://user-images.githubusercontent.com/49281851/185671260-4a447f6c-adf7-449f-8754-1ac4c93ef007.png)

1. Shared Memory
2. Message passing

An operating system can implement both methods of communication. Communication between processes using shared memory methods of communication and then message passing. Communication between processes using shared memory requires processes to share some variable, and it completely depends on how the programmer will implement it. One way of communication using shared memory can be imagened like this:
Suppose process1 and process2 are executing simultaneously, and they share some resources or use some information from onother process. Process1 generates information about certain computations or resources being used and keeps it as a record in shared memory. When process2 needs to use the shared information, it will check in the record stored in shated memory and thake note of the information as a record from another process as well as for delivering any specific information to other process.

## 1. Shared Memory Method

Ex: Producer-Consumer problem
There are two processes: Producesr and Consumer. The producer produces some items and the Consumer consumes that item. The two processes share a common space or memory location known as a buffer where the item produced by the Producer is stored and from which the Consumer consumes the item if needed. There are two versions of this problem: The first one is known as the unbounded buffer problem in which the Producer can produce upp to a certain number of items before it starts waiting for Consumer to consume it. First, the Producer and the Consumer will share soem common memory, then the producer will start producinf items. Tf the total produced item is equal to the size of the buffer, the producer will wait to get it consumed by the Consumer. Similarly, the consumer will wait for the Producer to produce it. If there are items available, Consumer will consume them.

```C
#define buff_max 25
#define mod %

    struct item{

        // different member of the produced data
        // or consumed data
        ---------
    }

    // An array is needed for holding the items.
    // This is the shared place which will be
    // access by both process
    // item shared_buff [ buff_max ];

    // Two variables which will keep track of
    // the indexes of the items produced by producer
    // and consumer The free index points to
    // the next free index. The full index points to
    // the first full index.
    int free_index = 0;
    int full_index = 0;

```

### Producer Process Code

```C

item nextProduced;

    while(1){

        // check if there is no space
        // for production.
        // if so keep waiting.
        while((free_index+1) mod buff_max == full_index);

        shared_buff[free_index] = nextProduced;
        free_index = (free_index + 1) mod buff_max;
```

### Consumer Process Code

```C
item nextConsumed;

    while(1){

        // check if there is an available
        // item  for consumption.
        // if not keep on waiting for
        // get them produced.
        while((free_index == full_index);

        nextConsumed = shared_buff[full_index];
        full_index = (full_index + 1) mod buff_max;
    }
```

In the above code, the Producer will start producing again when the `free_index+1` mod buff max will be free because if it it not free, this implies that there are still items that can be consumed by the Consumer so there is no need to produce more. Similarly, if free index and full index point to the same index, this implies that there are no items to consume.

## 2. Message Passing Method

In this method, processes communicate with each other without using any kind of shared memory. If two process p1 and p2 want to communicate, they proceed as follows:

- Establish a communication link(if a link already exists, no need to establish it again)
- Start exchanging messages using basic primitives.
  We need at least two primitives:
  - **send**(message, distination) or **send**(message)
  - **receive**(message, host) or **receive**(message)

![image](https://user-images.githubusercontent.com/49281851/185671348-cec0d037-1ec2-4cc5-87b0-f1016d5993c1.png)

The message size can be fixed size or of variable size. If it is of fixed size, it is easy for an OS designer but complicated for a programmer and if it is of variable size then it is easy for programmer but complicated for OS designer. A standard message can have two parts: **header and body**.
The **header part** is used for storing message type, distination id , source id, message length, and control information. The control information contains information like what to do if runs out of buffer space, sequence number, priority. Generally, message is sent FIFO style.

### Message Passing through Communication Link

#### Direct and Indirect Communication Link

While implementing the link, there are some quiestions that need to be kept in mind:

1. How are links established?
2. Can a link be associated with more than two processes?
3. How many links can there be between every pair of communicating process?
4. What is the capacity of a link? Is the size of a message that the link can accommodate fixed or variable?
5. Is a link uniderectial or bi-directional?

A link has some capacity that determines the number of messages that can reside in it temporarily for which every link has a queue associated with it which can be of zero capacity, bounded capacity, or unbounded capacity. In zero capacity, the sender waits untill the receiver informs the sender that is has received the message. In non-zero capacity cases, a process does not know whether a message has been received or not after the sen operation. For this, the sender must communicate with the reveiver explicitly. Implementation of the link depends on the situation, it can be either a direct communication link or an in-direct communication link.
**Direct Communication Link** are implemented when the processes use a specific process identified for the communication, but it is hard identify the sender ahead of time.
_Example: print the server_
**In-direct Communication** is done via a shared mailbox (port), which consist of queue of messages. The sender keeps the message in mailbox and the receiver picks them up

### Message Passing through Exchanging the Messages.

#### Synchronous and Asynchronous Message Passing:

A process that is blocked is one that is waiting for some event , such as resource becoming available or the completion of an I/O operation. IPC is possible between the processes on same computer as well as on the processing running on different computer i.e. in networked/distributed system. In both cases, the process may or may not be blocked while sending a message or attempting to receive a message so message passing may be blocking or non-blocking. Blocking is considered **synchronous** and **blocking sen** means the sender will be blocked untill the message is received by receiver. Similarly, **blocking receive** has the receiver block untill a message is available. Non-blocking is considered **asynchronous** and Non-blocking send has the sender sends the message and continue. Similarly, Non-blocking receive has the receiver receive a valid message or null. After a careful analysis, we can come to a conclusion that for a sender it is more natural to be non-blocking after message passing as there may a need to send the message to different processes. However, the sender expects acknowledgement from the receiver in case the send fails. Similarly, it is more natural for a receiver to be blocking after issuing the receive as the information from the received message may be used for further execution. At the same time, if the message send keep on failing, the receiver will have to wait indefinitely. That is why wi also consider the other possibility of message passing, There are basically three preferred combinations:

- Blockin send and blocking receive
- Non-blocking send and Non-blocking receive
- Non-blocking send and Blocking receive(Mostly used)

**In Direct message passing**, The process which wants to communicate must explicitly name the recipient or sender of the communication. e.g **send(p1, message)** means send message to p1. Similarly, **receive(p2, message)** means to receive the message from p2. In this method of communication, the communication link gets established automatically, which can be either uniderectional or biderectional, but one link can be used between one pair of sender and receiver should not possess more that one pair of links. Symmetry and asymmetry between sending and receiving can also be implemented i.e. either both processes will name each other for sinding the message and there is no need for the receiver for naming hte sender for receiving the message. The problem with this method of communication is that if the name of one process changes, this method will not work.

**In Inderect message passing**, processes uses mailboxes( also referred to as ports ) for sending and receiving messages. Each maibox hasa uniquie id and processes can communicate only if they share a mailbox. Link established only if processes share a common mailbox and a single links can be associated with many processes. Each pair of processes can share several communication links and these links may be unidirectional or bi-directional. Suppose two processes want to communicate through Inderect message passing, the required operations are: create a mailboxm use this mailbox for sending and receiving messages, then destroy the mailbox. The standard primitives used are: **send(A message)** which means send the message to mailbox A. The primitive for the receiving the message also works in the same way e.g **received (A, message)**. There is a problem with this mailbox implementation. Suppose there are more two processes sharing the same mailbox and suppose the process p1 sends a message to the mailbox, which process can share a single mailbox or enforcing that only one process is allowed to execute the receive at givent time or select any process randomly and notify the sender about reciever. A mailbox can be made private to a single sneder/receiver pair and can also be shared between multiple senders and a single receiver. It is used in client/server applications(in this case the server is the receiver). The port is owned by the receiving process and created by OS on the request of the receiver process and can be destroyed either on request of the same receiver processor when the receiver terminates itself. Enforcing that only one process is allowed to execute the receive can be done using the concept of mutual exclusion. **Mutex mailbox** is created which is shared by n process. The sender is non-blocking and sends the message. The first process which executes the receive will enter in the critical section and all other processes will be blocking and will wait.
Now, let’s discuss the Producer-Consumer problem using the message passing concept. The producer places items (inside messages) in the mailbox and the consumer can consume an item when at least one message present in the mailbox.

**_Producer Code_**

```C
void Producer(void){

    int item;
    Message m;
    while(1)
        receive(Consumer, &m);
        item = produce();
        build_message(&m , item ) ;
        send(Consumer, &m);
    }
}
```

**_Consumer Code_**

```C
void Consumer(void){

    int item;
    Message m;

    while(1){

        receive(Producer, &m);
        item = extracted_item();
        send(Producer, &m);
        consume_item(item);
    }
}
```

**Examples of IPC systems**

1)Posix : uses shared memory method.
2)Mach : uses message passing 3) Windows XP : uses message passing using local procedural calls

**Communication in client/server Architecture**:
There are various mechanism:

- Pipe
- Socket
- Remote Procedural calls (RPCs)
  The above three methods will be discussed in later articles as all of them are quite conceptual and deserve their own separate articles.

**References**:

1)Operating System Concepts by Galvin et al. 2) Lecture notes/ppt of Ariel J. Frank, Bar-Ilan University

[More Reference:](http://nptel.ac.in/courses/106108101/pdf/Lecture_Notes/Mod%207_LN.pdf)
[Video](https://www.youtube.com/watch?v=lcRqHwIn5Dk)
