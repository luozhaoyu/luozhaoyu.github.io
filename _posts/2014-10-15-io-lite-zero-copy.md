---
layout: post
title: "I/O-lite, Zero-copy"
---

### Design
* allow all subsystems and applications share the same buffer
* use immutable I/O buffers
    * avoid synchronization
* create an Abstract Data Type layer to represent I/O data
* provide ACL


### Reviews
* [Zero-copy] (http://en.wikipedia.org/wiki/Zero-copy)
* [Linux sendfile] (http://linux.die.net/man/2/sendfile)
* [Zero-copy in Linux] (http://www.ibm.com/developerworks/cn/linux/l-cn-zerocopy2/)

#### disadvantage
* Is there any follow-ups?
* Relations with DMA, zero-copy, mmap technologies?
