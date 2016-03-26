---
layout: post
title: "Review of First-Class User-Level Threads"
categories: blog
---

### two minutes warning
* User-level thread packages can set the actual duration of the warning by writing a value in the virtual processor data structure to avoid acquiring a spin lock near the end of the virtual processor quantum
* if an about-to-be-preempted thread is working on an important computation that needs to be continued on another physical processor, two-minute warning handler can save the state of the thread and then send an explicit interrupt to another virtual processor in the same address space, prompting it to migrate and run the preempted thread


### Reviews
* Kernel threads need not to be associated with a process
* M:N model performs well in an application are synchronizing with each other, ie taking local mutexes

#### [Threads](http://www.cs.uic.edu/~jbell/CourseNotes/OperatingSystems/4_Threads.html) Comparisions
* Processes are typically preemptively multitasked
* A kernel thread is a "lightweight" unit of kernel scheduling. Each process has >= 1 kernel thread. Kernel threads are preemptively multitasked if the operating system's process scheduler is preemptive. Kernel threads take much longer than user threads to be swapped
* User threads cannot take advantage of multithreading or multiprocessing and get blocked if all of their associated kernel threads get blocked even if they are ready to run
    * [green threads](https://en.wikipedia.org/wiki/Green_threads) are threads that are scheduled by a virtual machine
    * green threads must use asynchronous I/O to prevent blocking caused by only one single thread
    * green thread could normally use only one processor
* [Fiber](https://en.wikipedia.org/wiki/Fiber_(computer_science)) is a particularly lightweight thread of execution. Fiber use co-operative multitasking while threads use pre-emptive multitasking
    * Fibers are implicitly synchronized, yield
    * Fibers are being used to soft block themselves to allow their one threaded parent programs to continue working until related i/o operation event occurred


### Notes
* OS world| Psyche| Linux| Windows
* address space| address space| process| process
* kernel schedulable thread| virtual processor| light-weight process| thread
* user schedulable thread| thread| thread| fiber
* LWPS <= Threads = m : n, user schedulable thread is cheap
* how to creat a LWP in Linux? put it in blog


* what is the current threading model(Linux clone, Windows)?
about paper: do average, minimum value(noise), how about fail? lmbench starts here.
lowest value
time_value subtract may not be precise
check return value
UDP between hosts, different MTUs
comment on large amounts
keep simple
log scale
label graph seat
copy is evil, terrible
latency is more stressful
