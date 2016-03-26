---
layout: post
title: "U-Net, DCTCP"
categories: blog
---
### Why this paper
* How to improve performance and flexibility of netwroking layer performance?
    * move parts of the protocol processing into user space
    * remove kernel completely from the critical path and allow the communication layers used by each process to be tailored to its demands
    * the entire protocol stack should be placed at user level
    * OS and hardware should allow protected user-level access directly to the network

* Intents:
    * provide low-latency communication in local area settings
        * `processing_overhead = transmission_time + buffer_management + message_copies + checksum + flow-control + interrupt overhead + network_interface_control`
    * exploit the full network bandwidth even with small messages
    * faciliate the use of novel communication protocols

### U-Net
* each process has the illusion of owning the interface to the network
* Model: LogP (costs of communication)
    * sender: cost to send message `Os`
    * latency `l`
    * receiver: cost to receive message `Or`
    * `Time = 2l + 2Os + 2Or` assume `Os ~~ Or`: `Time = 2l + 4O`

#### History
* Distributed Computing 80s
    * early ethernet, bus-like, communication one-at-a-time, limits **scale**
    * Hadoop reshuffle would be a very bad idea
* Massively Parallel Processors early 90s (MPPs) "culler, parallel computer architecture"
    * scalable networks, tree-base switched net
    * high throughput, low latency (potential)
* Active Messages 92
    * Lightweight RPC (core idea similar RPC, but no compiler)

#### Philosophy
* bare minimal network layer
* virtualized network interface
    * protection: because multiple users of network
    * multiplexing/ sharing
    * usually done by OS, but here is done by **user level**
        * OS resource is limited, unified and constrained
        * performance gain
        * flexibility gain
* flexibility
* high performance

#### Basics
* EndPoints: queues: memory for data (communication segment, pinned host memory)
    * send queue, recv queue, free queue
        * queue contain descriptors: info about a message (e.g., destination addr, point to data (communication segment))
* Send
    1. form message in communication segment
    - push descriptor onto send queue
* Network interface (or U-Net)
    * must recognize that exists a message on send queue
    * once sent, mark descriptor bit
* Receive
    * Network Interface (NI): must demultiplex which is EndPoint
        * tag
    * Where to place message?
        * receive data into Free queue descriptor
        * put Free Queue -> Recv Queue
    * Notification
        * poll
        * event-driven (interrupt, callback)

#### Implementation
* Base Arch:
    * limited (small) communication segment
    * **pinned** in all memory (can not swap memory out)
        * but whole memory is limited
        * in order to let NI to access memory
    * small message optimization
        * put it directly into descriptor
* Direct Arch:
    * removes limits of communication segment (can send/ recv from any virtual address)
    * NI have to do DMA
* Kernel-emulated endpoints
    * because resource is limited
    * one OS EndPoint allows many EndPoints to map onto Kernel EndPoint
    * to migrate application to here

##### Fore SBA-200
* Network Interface has CPU, memory, CRC
    * EndPoint: Queues: where to place?
        * NI memory or Host memory
            * send, free queue in NI memory
            * recv queue in host memory


### The U-net paper discusses the notion of zero copy vs. true zero copy. What is the point being made here? Is this important for high performance applications and services?

#### Review
* U-Net has two levels of sophistication
    * *base-level* is *zero copy* requires an intermediate copy into a networking buffer
    * *direct acess* is *true zero copy*, which has no intermediate buffering

#### Comment
* True zero copy is useful, since the *processing overhead* consists of transmission time, buffer management, message copies, checksum, flow control, interrupt, network interface control and so on. Meanwhile, the speed of NIC < Memory < CPU. So it is natural to think about reducing the memory accessing times when there is almost no room for accelerate NIC speed. When transferring large bulk data, think about 1GB, it is very uneconomic to copy it once.
* However, buffer could be beneficial if the data to send is yieled continuously. e.g., the user process put only 1 Byte everytime for 1000 times to send 1K data. A buffer would largely increase the **throughput** and reduce **network congestion**. Even though the author preaches that his design has minimum overhead, it is still necessary to use *zero copy* to buffer a large packet when one does not need too much realtime communication.
* Modern high performance server such as Nginx has adopted similar mechanism, which is `sendfile()` a syscall since Linux 2.2. The main idea of `sendfile()` is to combine the `read()` and `write()` together, so that one does not need to copy some content from a file descriptor to user buffer and then write the buffer out. This idea tries to achieve **zero-copy** in kernel level, which would accelerate rending static file from disk to client much quickly.


### [DCTCP](https://en.wikipedia.org/wiki/Explicit_Congestion_Notification#DCTCP)
* [TCP Performance](http://www.cisco.com/web/about/ac123/ac147/ac174/ac196/about_cisco_ipj_archive_article09186a00800c8417.html)
