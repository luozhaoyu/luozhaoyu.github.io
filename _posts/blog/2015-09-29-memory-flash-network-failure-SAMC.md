---
layout: post
title: "SAMC, memory, flash, networks failure"
---

### [SAMC] (http://blog.acolyer.org/2015/03/25/samc-semantic-aware-model-checking-for-fast-discovery-of-deep-bugs-in-cloud-systems/)
* Use the knowledge of a system to craft the semantic could help to alleviate the state-space explosion in test cases
* Future test works would highly relate to the pattern extraction, since it is still manually so far. And of course, we do not need low-level test engineer that much
    * Future test engineers need to learn how to model the codes into a specific protocol, then generate tests from these protocols
        * This kind of test methodology seems to be more precise than fuzzy test and could give more understanding of why the codes fail


### A Large-Scale Study of Flash Memory Failures in the Field
* Nowadays, system designers do not need to worry about read disturbance, since the modern SSD has mitigated the problem of reading operation which breaks neighbour blocks
* I am also think temperature is not a problem, since:
    * Technician in data-center or hardware engineer may come up a better solution to lower the temperature
    * I doubt the temperature sensor has not been widely supported by the OS, so that we could not take it into account

#### Background
What are they looking at?

* host-correctable errors: Logical correction could assist in SSD ECC uncorrectable
* snapshot of SSD

Flash chips

* cells: storing information (as level of charge)
* faster, more costly
    * SLC: 1 or 0
    * MLC: 0, 1, 2, 3
* block: some what large ~ 256K
    * subdivided into pages: ~ 2K, 4K
* flash-based SSD
    * interface: block-based read/write
* Flash Translation Layer (FTL) flash controller
    * How to do Parallelism?
    * Performance? Erase is very slow
    * Reliability?
        * **wearout**, repeated program/erase circles make a block unusable over time
        * disturbance, may flip neighbouring cells (page next door) when reading

read/write

* read (unit: page)
* write
    * erase (unit: block)
    * program (unit: page)

#### Solution
* [Log-structured design] (https://en.wikipedia.org/wiki/Log-structured_file_system)
    * program at end of log
    * introduce new problem: **Mapping Info**: Logical -> Physical translations
        1. how to persistent? since it is in memory
            * cheap SSD would scan whole and build index
        - size: big SSD needs bigger storage
        - how to handle garbage (GC)

#### Graphs/Tables
Hypothesis: SSD use everything firstly, then mark bad blocks, which lower failure rate


### Memory Errors in Modern Systems
* If the system is not using stronger ECC, like chipkill-correct ECC, you have nothing to prevent from the system failure. So one thing system designer should consider is how to recover from the fault-prone system, i.e., fail as soon as possible.
    * So one thing you do not need to worry is to how to prevent system crash from memory error. If there is memory error, just let it goes

### Understanding Network Failures in Data Centers: Measurement, Analysis, and Implications
* The paper does not prove the invalidity of redundancy network. Since all the problems come from either the wrong configuration of back-up network or from bugs in fail-over mechanism. This is totally the problem of the software
    * So I think the worry of the efficiency of network redundancy is least important. We still need redundancy network
* Another thing is the price or model of the ToR, it does not affect network performance too much
