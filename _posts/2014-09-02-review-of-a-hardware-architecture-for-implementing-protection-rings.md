---
layout: post
title: "Review of A Hardware Architecture for Implementing Protection Rings"
---

### Introduction
Mechanisms are applicable to any computer system which uses segmentation as a memory addressing scheme.

### Access Control in a Computer Utility
* Controlling access to stored information is the most popular
* Four Criteria: functional capability, economy, simplicity, programming generality

### [Segmented Virtual Memory Environment](http://en.wikipedia.org/wiki/Virtual_memory#Segmented_virtual_memory)
* SDW: segment descriptor words, each SDW can describe a single segment in the virtual memory
* descriptor segment: an array of SDW
* DBR: descriptor base register

### Controlling Access in a Segmented Virtual Memory
#### Three specific assumptions true of Multics
1. new process created when user login
    * username is associated with the process
    * active agent & only means of referencing and manipulating on-line information
- on-line storage is a collection of segments. A process can reference a segment of on-line storage only if the segment is first added to the virtual memory of the process
- users that are premitted to access each segment are named by an access control list associated with each segment

#### Finer control
* Finer control on access is required once a segment is included in the virtual memory
* SDW contains finer constraints
* Flags are natural constraints, make a segment the smallest protected unit

#### Why Multics limited the number of domains(Protection Rings)
* Access capability would vary as the execute point change
* domain: a set of access capabilities. A single process needs multiple domains. *This idea is difficult to implement*

### Protection Rings
* flag OFF: accessibility is not in any ring of the process
* Gates: the start of exeuction in a particular domain to certain program locations
* Gate extension(in each SDW): specifies the consecutively numbered rings above the execute bracket of the segment
* upward ring switch is unrestricted
* In implementation, top of the write bracket = bottom of the execute bracket

### Call and Return
#### hardware does not implement upward calls and downward returns without software intervention
##### Problems of called procedures
1. called procedure must have a way of finding a new stack area without calling procedure
    * A fixed word of each stack segment
    * Processor provides the stack segment number
- called procedure must have a way of validating references to arguments to prevent from being tricked into accessing inaccessible argument
    * let the called procedure runs as though in a higher ring
- called procedure must have a way of knowing it would not return to a ring higher than the calling procedure
    * let the processor leave the ring number in an accessible register

##### Problem Upward call in returning back
1. calling procedure may specify inaccessible arguments(HARDWARE solution is straightforward)
    * require calling procedure specify only accessible arguments(compromises programming generality)
    * dynamically include in the ring of the called procedure(compromises programming generality, expensive)
    * copy arguments to accessible segments(restricts parallel processes)
- a gate must be provided for the downward return

HARDWARE solution: generating a trap to a supervisor procedure which performs the necessary environment adjustments, which are upward call and downward return

### Hardware Implementation of Rings
* SDW(SDW.R1, SDW.R2, SDW.R3) each is 3-bit and flags(SDW.R, SDW.W, SDW.E) is single-bit
    * write: [0, SDW.R1]
    * execute: [SDW.R1, SDW.R2]
    * read: [0, SDW.R2]
    * gate extension: [SDW.R2+1, SDW.R3]
* SDW.GATE, single fixed-length, beginning at location 0 of a segment, the list of gate locations of a segment is compressed here(requiring all gate locations to be gathered together), contains the number of gate locations present
* IPR: instruction pointer register, specifies the current ring of execution and the two-part address of the next instruction to be executed
* TPR: temporary pointer register, to form the two-part address of each virtual memory reference, program inaccessible

#### Rings implementation
* access checking logic, to validate each virtual memory reference(how to describe: trace the processor instruction cycle)
* special instructions for changing the ring of execution

#### Processor Instruction Cycle
1. retrieving the next instruction to be executed
- calculating in TPR the effective address of the instruction's operand
* perform the instruction
    * do read
    * do write
    * does not reference operand(so no access validation is required)
        * EAP-type instruction, Effective Address to Pointer Register, load the RING, SEGNO, WORDNO fields of PRn, the only way to load PR's
        * other transfer instruction, though needs not validation, an advance check is performed before reloading IPR from TPR
        * call and return instructions, software intervention would occur when perform an upward call or downward return
            1. CALL must be directed at a gate location, even the called procedure has the same ring
                * this provides protection against accidental calls to locations that are not entry points

#### other considerations
1. trap, processor would change the ring to 0 and transfer control to a fixed location in the supervisor when trap detected
- privileged instructions(load DBR, start I/O, restore processor state) could only be executed in ring 0

### Call and Return Revisisted
* arguments. All arguments >= caller ring
* a return to the proper ring is accomplished. RETURN instruction is guaranteed to generate an effective ring number >= calling procedure

### Use of Rings
* Implicit invocation of certain ring 0 supervisor procedures: trap
* Explicit invocation of ring 0 1 supervisor procedures by procedures in rings 2~5: to gates by standard subroutine calls
* Procedures in rings 6 7 are inaccessible to supervisor gates

### Conclusions
The hardware mechanisms solves three problems in a system, which equip with supervisor/user protection scheme and a shared virtual memory based on segmentation

* users can create arbitrary, but protected, subsystems for use by others
* the supervisor can be implemented in layers which are enforced
* the user can protect himself while debugging his own(or borrowed) programs *debug in a higher ring*

Benefits of Protection Rings

* let designer to implement his own system or subsystem
* hardware implementation requires very small additional costs in hardware logci and processor speed
* make it possible of calling protected subsystems or other procedures to use a same mechanism(let the developer gets rid of the idea that supervisor call is expensive)

### Reviews
* Modern Linux is using virtual memory management. When process is swithed, the process's TLB would be loaded into MMU's TLB. If the process specifies an inaccessible address, the MMU would raise an interrupt to notify the OS
* Everything is a segment(on-line storage)
* what is WORDNO(offset)? descriptor segment(Global Descriptor Table)? DBR(GDT register)?
* what is PR(pointer register), IPR and TPR, two-part address?

### Notes
* paraphrase: delegation "trust" between layeres. TRUST, what is the boundary
* Computer Utility=OS, Supervisor Program=kernal, IPR=PC Instruction Pointer, Segment Descriptor=Segment Table, SDW=Segment Table Entry
* [ROP] (http://en.wikipedia.org/wiki/Return-oriented_programming)
