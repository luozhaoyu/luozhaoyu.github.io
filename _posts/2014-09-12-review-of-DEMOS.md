---
layout: post
title: "Reviews of DEMOS"
---

### DEMOS
* links provide both message paths and optional data sharing between tasks
* Switchboard task is provided to allow mutually consenting arbitrary tasks to communicate


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
* links are pipes, channels are sockets?


### Notes
about paper: do average, minimum value(noise), how about fail? lmbench starts here.
