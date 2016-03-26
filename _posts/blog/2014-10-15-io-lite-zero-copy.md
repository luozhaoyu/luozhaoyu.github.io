---
layout: post
title: "I/O-lite, Zero-copy"
categories: blog
---

### Design
* allow all subsystems and applications share the same buffer: create an Abstract Data Type layer to represent I/O data
    * each process uses the same virtual address for simplification
        * so the number of stacks is limited
    * use immutable I/O buffers
        * avoid synchronization
    * how-to share memory? [Capability-based addressing](http://en.wikipedia.org/wiki/Capability-based_addressing)
* IPC mechanisam: [page remapping](http://linux.die.net/man/2/mremap) & shared memory
* provide ACL

### Notes
* Why networking guy like use [Gather-scatter](http://en.wikipedia.org/wiki/Gather-scatter_(vector_addressing))?
    * [Vectored I/O](http://en.wikipedia.org/wiki/Vectored_I/O)

### Reviews
* [Zero-copy](http://en.wikipedia.org/wiki/Zero-copy)
* [Linux sendfile](http://linux.die.net/man/2/sendfile)
* [Zero-copy in Linux](http://www.ibm.com/developerworks/cn/linux/l-cn-zerocopy2/)

#### disadvantage
* Is there any follow-ups?
    * the kernel has become large and complicated, it is difficult to change, people are conservative
* Relations with DMA, zero-copy, mmap technologies?
