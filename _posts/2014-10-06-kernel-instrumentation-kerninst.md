---
layout: post
title: "Kernel Instrumentation, KernInst"
---

### [KernInst] (http://www.dyninst.org/sites/default/files/downloads/w2003/KerninstAPI.pdf)
structure

* kerninstd: user-level daemon
    * allocate the patch area heap
    * parse the kernel's runtime symbol table
    * obtain permission to write to any portion of the kernel's address space
    * call /dev/kerninst to allocate kernel memory to code patches
    * use kernel machine code to build a control flow graph
* /dev/kerninst: runtime-loaded KernInst driver
    * communicate with kerninstd via file operations
    * installed on-the-fly
* run-time allocated heaps:
    * patch area heap: dynamically generated code. it is mapped by *mmap*, so it is accessible to kernel and kerninstd
    * timers and counters heap: used when instrumentation code contains performance gathering annotations

* Overwrting a single instruction is safe than multiple-instruction splicing
    * beacuse a thread execute a mix of the pre-slice and post-slice sequences

### Structural Analysis
1. Runtime symbol table is parsed to determine the in-memory start of all kernel functions
- Each function's machine code is then read from memory and parsed into basic blocks

### Use free registers at instrumentation points
* for jumping/executing to dynamically inserted code
* run interprocedural live register analysis algorithm

### Code Splicing
* inserting runtime generated code before a desired kernel code location (the instrumentation point)
* Kerninstd splices by overwriting the instrumentation point instruction with a branch to patch code, which includes:
    * dynamically generated code
    * original overwitten instruction
    * jump back to the instruction stream after original instrumentation point

### Springboards

### Reviews
* [Symbol table] (http://en.wikipedia.org/wiki/Symbol_table)
* [Hooking] (http://en.wikipedia.org/wiki/Hooking)
* [Control flow graph] (http://en.wikipedia.org/wiki/Control_flow_graph)
* [Just-in-time compilation] (http://en.wikipedia.org/wiki/Just-in-time_compilation)
* Why Solaris?
    * Solaris kernel is more clean
    * Solaris could track the source of a trap
* Why not let the compiler to patch?
