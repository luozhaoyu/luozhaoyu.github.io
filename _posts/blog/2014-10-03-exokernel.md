---
layout: post
title: "Exokernel"
categories: blog
---

### Introduction
* A small kernel securely exports all hardware resources through a low-level interface to untrusted library operating systems
* The single overriding goal: to separate protection from management -> to provide an interface that is as low-level as possible, the approaches:
    * give each application its own virtual machine
    * exokernel way: export hardware resources rather than emulating
* three techniques to export resources securely:
    1. secure bindings
    - visible resource revocation
    - abort protocol

### Secure Bindings
* Secure binding is a protection mechanisam that decouples authorization from the actual use of a resource
* Secure binding allows the kernel to protect resources without understanding them

How it improves performance?

* protection checks are simple and quick
* a secure binding performs authorization only at bind time

How to implement it?

1. hardware mechanisms
- software cachiing
- downloading application code
    * the code is invoked on every resource access or event to determine ownership and the actions that the kernel should perform

### Multiplexing Physical Memory
* a secure binding is created when a library operating system allocates a physical memory page
* the principle of a page-table interface: priviledged machine operations such as TLB loads and DMA must be guarded by an exokernel
    * the page table should be visible(read only) at application level (is it a drawback?)
* how to break a secure binding of physical memory: an exokernel would flush all TLB mappings and any queued DMA requests

### Aegis: an Exokernel
#### Processor Time Slices
* a long-running scientific application could allocate contiguous time slices in order to minimize the overhead of context switching


#### Dynamic Packet Filter
* the network subsystem uses aggressive dynamic code generation techniques to provide efficient message demultiplexing and handling



### Reviews
* [Exokernel](http://wiki.osdev.org/Exokernel)
* [Capability-based security](http://en.wikipedia.org/wiki/Capability-based_security)
* [End-to-end principle](http://en.wikipedia.org/wiki/End-to-end_principle)
* [MIT Exokernels slides](http://pdos.csail.mit.edu/exo/exo-slides/sld001.htm)
* [paper QA](http://c2.com/cgi/wiki?ExoKernel)

### Notes
Cons:
* DPF is not enough for protecting packets from intercepting by other applications, since any application could access the network interface
* The size of a time slice has not been decided
* Kernel should judicates over different VMs, we need one more layer to decide the quotas(like Xen)


* Preview next paper: current context & detail of how to performance
