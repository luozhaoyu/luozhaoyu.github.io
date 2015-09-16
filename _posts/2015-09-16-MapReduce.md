---
layout: post
title: "MapReduce"
---

### MapReduce
* M/R programming model
* Execution isssues, run time problems
* Scheduling and fairness across jobs
    * greedy
    * fair scheduler (slot-based)
    * flow-based fair scheduler

* large volumes of data distributed across many machines, process this data to get derived data

* Map (k, v): outputs (k, v) pairs organized by k
    * `file -> "w, 123"`
* Reduce (k, v) pairs output by map
    * for the same k, compute some aggregate over all values for key
    * `"w", frequency`

#### MR spec:
1.a list of input files of "splits"
    * 16 ~ 64 MB <= |B|
    * number of splits == number of mappers
- format of input, pattern
- name of the mapper routine
- output file space
- number of reduce tasks
- format of output
- reducer routine
- maximum number of machines
- maximum resource for M/R

How to decide the number of M or R? not easy to decide

#### Master (job manager)
* schedule reducers on-the-fly as its input is ready
* wait for number of maps to finish
1. allocate Maps (locality)
    1. reads split
    - passes (k, v), sort
    - mapper writes to local (files) that are key space partitioned
    - informs master when done
- Reduces (completely random)
    1. master provides locations of files with initial data
    * reduce workers use tcp/http to retrieve
    * reads all input from all mappers sorts by k and feeds all record for a key to reduce()
    * written to a local tmp file
    * rename to a global FileSystem file
    * reduce reports to master

### Run-time
1. workers may fail
    * reducers: recompute that in-progress failed tasks
    * mappers: all tasks of that worker are recomputed even though it is completed
* master fail: re-run the whole task
- dealing with stragglers:
    * fail end of a job, duplicate all tasks


### Why there are outliers
* machine difference
    * busy, bad: output is garbled -> recomputations
    * tasks contending
    * solution:
        * duplicate (resource use)
        * kill and restart (queue delay)
            * tnew < tcurrent, otherwise, it is not worthwhile to kill and restart
* network performance, cross-rack traffic (needs network aware placement)
    * shared resource contention
    * data locality is not satisfying
    * reducers self-interfering on the same job
* MapReduce partition imbalance, data skew
    * schedule tasks with more data earlier
    * multiple jobs needs affinity to nodes, racks, cluster
        * each of it has its own queen
            * pick a task at head -> greedy: but it could not ensure starvation
            * look at jobs that are far from fair quota ([sticky-slot problem] (http://people.csail.mit.edu/matei/talks/2009/msr_mapreduce_scheduling.pdf))
                * delay-scheduling: let the worker starve for a while to see if there is chance to find in-rack task, if wait number exceeds certain threshold, let this worker works

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
