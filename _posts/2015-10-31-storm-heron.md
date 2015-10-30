---
layout: post
title: "Storm, Heron"
---

### Storm / Heron
* one process is one JVM
* one JVM has multipe threads (executors)
* one thread has multipe tasks

### Approximate operators
* two ways:
    * sampling
	* [sample-and-hold] (https://en.wikipedia.org/wiki/Sample_and_hold): heavy-hitter
	    * identify subset of elements, e.g., give me 5% of tweets 
    * sketching (bloom filter)
	* [loglog operator] (http://blog.notdot.net/2012/09/Dam-Cool-Algorithms-Cardinality-Estimation): estimating cardinality of a multi-set
	    * bit vector
	    * index = compute(hash(x))
	* [count min-sketch] (https://en.wikipedia.org/wiki/Count%E2%80%93min_sketch): estimating frequency counts

#### Common
* Topology: interconnection between spout and bolt
* each spout and bolt could map to multiple tasks
* each stream is an indefinite sequence of tuples (tweets)
* support:
    * **at most once**: TCP
    * **at least once**: receive acks from multiple tuples, if timetout, replay tuple from spout

#### Different
* Storm: there is no resource isolation between different tasks
    * Heron: each container runs the same task (for debug visibility)
* backpressure: Heron would automatic adjust network throughput to prevent from making no progress
    * one of task fails, entire tweet needs to be restarted

#### Failure
* on failure, replay tweet by tweet from start, but spark streaming recover from intermediate RDD
* nodes failure: Spark could do some recovery
  
#### Zookeeper
* where to store the state of one tweet?
    * paper does not say
* keeping runtime state: nimbus and supervisor

### Reference
* [PDX DevOps- Stream processing, Heka and Riemann] (http://www.slideshare.net/nickchappell/pdx-devops-stream-processing-heka-and-riemann)