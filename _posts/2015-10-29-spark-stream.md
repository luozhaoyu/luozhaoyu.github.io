---
layout: post
title: "Spark Streaming"
---

### Current distributed stream processing model
* requiring hot replication
* long recovery times
* do not handle stragglers
* Continuous operator model: Storm, TimeStream, MapReduce Online and streaming DBs
    * long-running, stateful operators receive each record, update internal state and send new records
	* since it is stateful, it is hard to provide fault tolerance efficiently
    * like Map and Reduce job, no start and no stop, always running
    * how to do recovery?
        * replication and also **synchronization protocol**
	    * to ensure replicas receive messages in the same order
	* upstream backup: upstream nodes buffer sent messages and replay them to a new copy of a failed node
	    * this will take longer time to recover: the whole system needs to wait for a new node to serially rebuild the failed node's state by rerunning

### Stream Analytics
* Real-time fraud detection / Spam compaigns
* Analytics of web services
* Billing, Advertisers
* Stream of data -> temporal analysis (within 1~2s) + comparison against a model
* Continuous queries
* Intensive FaultTolerance at sacle (O(100))

### Frameworks
* Data Parallel
    * custom-built for stream analytics (storm/heson/millwheel)
    * batch systems tuned into steam analytics (SparkStream/MRonline)
* Parallel DB
    * Aurora / Borealis, single node streaming DB

### Drawback of MapReduce
* MapReduce: mappers informs JobTracker, JobTracker call Reducer, Reducer pulls the result from Mapper **after** Mapper finish
* Stream <w, 1> individually, pipe it into downstream Reducer
    * there is no fault tolerance here
    * too much packet over network
* How about: Mapper spill files as chunks of memory are filled, mapper pushes to appropriate reduce
    * map fails:
        * persist spill locally to HDFS
	* prepare 2x mappers to perform tasks
	    * it is wasteful at most time
	    * would not help with straggler mapper
    * reduce fails:
        * restart reducer, and read data from mapper again

### Spark Streaming
* divide continuous job into multiple small interval jobs
    * the state at each timestep fully deterministic given the input data: no need for synchronization protocol
    * dependencies between two states are smaller
* short, stateless, deterministic tasks
* reduceByKey: stateless operator
* reduceByWindow: stateful operator
* **exactly once processing** (storm consistency model: **at most once** or **at least once**)

### Failure: parallel recovery
* When a node fails, each node in the cluster works to recompute part of the lost node's RDDs: no cost of replication
    * handle stragglers: can recover from stragglers using speculative execution
* using **lineage** for recovery: a graph of deterministic operations


#### Scheduling
* data skew may happen in one partition (one interval)
* 一个节点坏了后,以及雪崩效应
