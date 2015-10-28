---
layout: post
title: "Spark Streaming"
---

### Stream Analytics
* Real-time fraud detection / Spam compaigns
* Analytics of web services
* Billing, Advertisers
* Stream of data -> temporal analysis (within 1~2s) + comparison against a model
* Continuous queries
* Continuous operator
    * like Map and Reduce job, no start and no stop, always running
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
* reduceByKey: stateless operator
* reduceByWindow: stateful operator
* **exactly once processing** (storm consistency model: **at most once** or **at least once**)

#### Scheduling
* data skew may happen in one partition (one interval)
* 一个节点坏了后,以及雪崩效应
