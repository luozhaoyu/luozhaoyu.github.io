---
layout: post
title: "MapReduce"
---

### Why there are outliers
* machine difference
* network performance, cross-rack traffic
* MapReduce partition imbalance, data skew

### Comments
* Mantri wants to do real-time analysis over the outliers
    * run task on other slave
    * use local network as much as possible
    * sort task by their sizes to reduce long task run time
* Were I design the system, I would use a queuing system to store all the splitted task, shuffled partitions, then treat all slave machines as workers. As long as we could divide whole task into a fine grained level, we could almost balance the task excution time.
    * Once queue is empty, it would notify the master its corresponding mapreduce phase is done
    * A problem is one single queue may not be big enough to hold all data in its memory, we need Kafka
    * Drawback is we would at least double the **network traffic** (send to queue and read from queue)
        * But I think it is tolerable, since the major outlier problem lies in MapReduce itself and heterogenous machines
        * The main idea is to balance the transmition power with compute power of one cluster
            * So a further idea is to assign half tasks directly into workers. i.e., combines the action **push** and **pull** together

### Reference
* [MapReduce数据处理性能分析] (http://zhangjunhd.github.io/2013/09/26/mr2.html)
* [Spark随笔（三）：straggler的产生原因] (http://www.cnblogs.com/zx247549135/p/3977725.html)
