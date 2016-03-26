---
layout: post
title: "Synthesis kernel"
---

### Design Goal
1. high performance
    * combine kernel code synthesis, which decreases kernel call overhead through specialization
    * reduce synchronization
- self-tuning capability to dynamic load and configuration changes
- a simple, uniform and intuitive model of computation with a high-level interface

### Methods to synthesize code
* Factoring Invariants: bypass redundant computations, like constant folding
* Collapsing Layers: eliminates unnecessary procedure calls and context switches
* Executable Data Structure: shorten data structure traversal time when the data structure is always traversed the same way

### Reduced Synchronization
* code isolation
* procedure chaining
* optimistic synchronization

### Threads
* TTE: thread table entry (just like process table in UNIX)
* when thread makes kernel call, the thread would execute in the kernel model (rather than has an associated kernel thread)

#### Context Switches
synthesis context-switch is shorter:
1. switch only part of the context
    1. do not save floating point co-processor, and an illegal trap happens when its first execution of floating point instruction
    - MMU address space switch
- use executable data structures to minimize the critical path
    * no "dispatcher" procedure in synthesis. ready-to-run threads are chained in an executable circular queue. context-switch-out thread would **jmp** directly to the next context-switch-in thread


* Each thread may have its own error trap handlers, and an error signal will be sent to the interrupted thread. Then the error signal handler would run in user mode
* Scheduling policy is round-robin with an adaptively adjusted CPU quantum per thread






### Reviews
* [self-modifying code] (http://en.wikipedia.org/wiki/Self-modifying_code#Massalin.27s_Synthesis_kernel)
* principle of frugality: use the least powerful solution to a given problem
* [constant folding] (http://en.wikipedia.org/wiki/Constant_folding)
* [common subexpression elimination] (http://en.wikipedia.org/wiki/Common_subexpression_elimination)
