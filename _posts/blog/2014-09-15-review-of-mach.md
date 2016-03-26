---
layout: post
title: "Review of MACH"
categories: blog
---

### vm_allocate
1. **User Process**call **Kernel** with vm_allocate()
- **Kernel**call **Pager** about page initialization
- **User Process** meets page fault interrupt
- **Kernel** request **Pager**
- **Pager** provide page
- **Kernel** resumes **User Process**

### IPC
* task, thread, port, message
* the concept of thread at time ensure the ability of concurrency

#### port
* likes FDs, each process has a port table
* representing each endpoint of a two-way IPC
* system call
    1. ask the kernel for access to a port
    - use IPC to send messages to that port
* model on UNIX file system concepts


### Virtual Memory
* copy-on-write starts from here


### Mach 3
* Mach becomes independent from BSD
* SO SLOW. a simple Unix system call would refer to tens of port, permission setting, message passing

### deficiencies
* kernel check every message for validity
* additional privileges would be granted to user-space programs
* overhead of IPC
* kernel has no clue about memory paging
* Mach bases on mapping memory around between programs, any "cache miss" made IPC calls slow


### Reviews


### Notes
