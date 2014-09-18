---
layout: post
title: "Reviews of DEMOS"
---

### DEMOS
* links provide both message paths and optional data sharing between tasks
* Switchboard task is provided to allow mutually consenting arbitrary tasks to communicate
* security is a big issue to Los Alamos


### DEMOS/MP
* A process's abilities are determined by the links that it holds. A process's privileges are determined only by the links that it holds
* not all machines have disks, remote paging is implemented

#### Switchboard
* Switchboard is a single name server, which aids in establishing connections between processes like a telephone operator. Any process with a link to the switchboard has to:
    - announce itself to the world
    - request a link to another process that has already announced itself
* Only one global switchboard. It's location is broadcast across the network.
* Local switchboard is an exact copy of the global switchboard
* Once a new machine is booted, it posts a link to the switchboard
* Machines are partitioned by multiple memory manager
* A centralized recorder reliably stores all messages that are transmitted

### Reviews
* channels are sockets? is just a way to provide multiplex
* will the switchboard and the centralized recorder be the whole system's bottleneck?
    * it would, if the number exceeds 1000
    * SB and recorder is fast, there will be hierachy SBs


### Notes
* everything is a link
* link is equivalant to file descriptor(stdin, stdout, switchboard, processmanager, memorymanager), link a handle to object
