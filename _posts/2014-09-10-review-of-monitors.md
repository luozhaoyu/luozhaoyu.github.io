---
layout: post
title: "Reviews of Monitors"
---

### [Monitors] (http://en.wikipedia.org/wiki/Monitor_(synchronization))


### Mesa
* each object has its own monitor
* let signal thread move to the queue and execute rather than exit and lose occupancy of the monitor
* Mesa's treatment of NOTIFY as a hint, what is hint?
    * Process could resume at convenient future time rather than immediately
    * applications need not to determine the exact conditions, they could specify a broader conditions
* Fundamental Deficiency: the faster process may have to wait for the slower one with two mutual exclusion processes on different processors?
#### JAVA
* java does not have explicit condition variable

### Reviews
* Naked Notify=Spurious Wakeup

### Notes
* busy waiting is evil: waste resources
