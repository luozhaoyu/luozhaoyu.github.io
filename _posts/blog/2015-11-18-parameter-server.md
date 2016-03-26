---
layout: post
title: "Parameter Server"
categories: blog
---

### ML challenges
* needs enormous network bandwidth
* cost of synchronization and machine latency is high: since ML algorithms are sequential
* fault tolerance is critical: machines and jobs can be preempted

### Parameter Server
* picking the right systems techniques, adapting them to the machine learning algorithms
* flexible consistency (seqeuntail, eventual, bounded delaj consistency)
* examples
* systems tricks/ optimizations
    * consistent hashing
* ParameterServer atop Tez,MR,Spark
    * Spark could also do the same thing (partition model, fault tolerance)
        * why not use Spark? is its graph more convenient?


#### data
* k,v -> memcached
* algo-specific
* algo-agnostic

#### Developer Advantages
1. enable application-specific code to remain concise
- provide robust, versatile, and high-performance ML implementation

#### Engineering Challenges
* Communication:
    * key-value pair is inefficient, worker would send a segment of a vector, or an entire row of the matrix (a part of object)
    * local filter + compression
* Fault tolerance: support dynamic scaling
    * replicating models, ack worker push only after replications (chain replication)
    

#### Risk minimization


### Reference
* [USENIX](https://www.usenix.org/node/186215)
* [talk PPT](http://parameterserver.org/talks/PSLong2014.pdf)