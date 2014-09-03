---
layout: post
title: "Review of A Hardware Architecture for Implementing Protection Rings"
---

Introduction
----------
Mechanisms are applicable to any computer system which uses segmentation as a memory addressing scheme.

Access Control in a Computer Utility
----------
* Controlling access to stored information is the most popular
* Four Criteria: functional capability, economy, simplicity, programming generality

[Segmented Virtual Memory Environment](http://en.wikipedia.org/wiki/Virtual_memory#Segmented_virtual_memory)
----------
* SDW: segment descriptor words, each SDW can describe a single segment in the virtual memory
* DBR: descriptor base register

Controlling Access in a Segmented Virtual Memory
----------
### Three specific assumptions true of Multics
- new process created when user login
    * username is associated with the process
    * active agent & only means of referencing and manipulating on-line information
- on-line storage is a collection of segments. A process can reference a segment of on-line storage only if the segment is first added to the virtual memory of the process
- users that are premitted to access each segment are named by an access control list associated with each segment

### Finer control
* Finer control on access is required once a segment is included in the virtual memory
* SDW contains finer constraints
* Flags are natural constraints, make a segment the smallest protected unit

### Why Multics limited the number of domains(Protection Rings)
* Access capability would vary as the execute point change
* domain: a set of access capabilities. A single process needs multiple domains. *This idea is difficult to implement*

Protection Rings
----------
* flag OFF: accessibility is not in any ring of the process
* Gates: the start of exeuction in a particular domain to certain program locations
* Gate extension(in each SDW): specifies the consecutively numbered rings above the execute bracket of the segment
* upward ring switch is unrestricted
* In implementation, top of the write bracket = bottom of the execute bracket

Call and Return
----------
### hardware does not implement upward calls and downward returns without software intervention
#### Problems of called procedures
- called procedure must have a way of finding a new stack area without calling procedure
    * A fixed word of each stack segment
    * Processor provides the stack segment number
- called procedure must have a way of validating references to arguments to prevent from being tricked into accessing unaccessible argument
    * let the called procedure runs as though in a higher ring
- called procedure must have a way of knowing it would not return to a ring higher than the calling procedure
    * let the processor leave the ring number in an accessible register

#### Problem Upward call in returning back
- calling procedure may specify unaccessible arguments(HARDWARE solution is straightforward)
    * require calling procedure specify only accessible arguments(compromises programming generality)
    * dynamically include in the ring of the called procedure(compromises programming generality, expensive)
    * copy arguments to accessible segments(restricts parallel processes)
- a gate must be provided for the downward return

HARDWARE solution: generating a trap to a supervisor procedure which performs the necessary environment adjustments, which are upward call and downward return

Hardware Implementation of Rings
----------

