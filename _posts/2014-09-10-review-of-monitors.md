---
layout: post
title: "Reviews of Monitors"
---

### [Monitors] (http://en.wikipedia.org/wiki/Monitor_(synchronization))


### Mesa
* each object has its own monitor
* Mesa's treatment of NOTIFY as a hint, what is hint?
    * Process could resume at convenient future time rather than immediately
    * applications need not to determine the exact conditions, they could specify a broader conditions
* Fundamental Deficiency: the faster process may have to wait for the slower one with two mutual exclusion processes on different processors?

### Reviews
* Naked Notify=Spurious Wakeup

### Notes
about paper: do average, minimum value(noise), how about fail? lmbench starts here.
