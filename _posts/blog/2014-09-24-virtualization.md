---
layout: post
title: "Virtualization"
categories: blog
---

### [Memory Resource Management in VMware ESX Server](https://www.usenix.org/legacy/events/osdi02/tech/waldspurger/waldspurger_html/esx-mem-html.html)
* double paging problem
    * use randomized page replacement policy, since paging will be uncommon
* ballooning, let guest OS invoke its own native memory management algorithms. If fails, continue to use paging mechanism
* shares-per-page ratio, relative resource acquisition right

### Xen and the Art of Virtualization
* [tagged TLB](http://blogs.bu.edu/md/2011/12/06/tagged-tlbs-and-context-switching/): there is ASID in every TLB entry
    * [address space identification](http://www.bottomupcs.com/virtual_memory_hardware.html#flushing_tlb): each address space(process) gets its own ID. When process context switch happens, the TLB needs not to be flushed, because two processes with a same virtual address would point to different physical address. And then TLB does not have to be invalidated on thread context switch as the ASID is unique

#### full virtualization drawbacks


#### design principles
* Support for unmodified application binaries with existing standard ABIs
* Support full multi-application operating systems
* Paravirtualization is key to high performance and isolation
* Hidding the effects of resource virtualization

