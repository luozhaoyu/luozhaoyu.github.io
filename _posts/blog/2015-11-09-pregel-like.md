---
layout: post
title: "Pregel-like Graph Processing Systems Comparison"
---

### Pregel-like
* [Bulk Synchronous Parallel model] (https://en.wikipedia.org/wiki/Bulk_synchronous_parallel)
    * is a vertex state machine (think like a vertex)
    * but may encounter the straggler problem
* use vertex-centric approach, graph parallel
    * each vertex runs parallel: gather sum apply scatter
* address in-memory batch processing of large graphs
* terminate when all vertices are inactive and no more messages are in transit
* computation is performed on locally stored data
* Pregel only supports graphs that fit in memory
* master/workers model: the master partitions the input graph into partitions and assign it to a worker
* Global state?
    * global aggregators
    * phases of the algorithm

### Storage/partitioning
* using HDFS as underlying storage and loading into memory
* Graphs:
    * hash-based
    * vertex-cut based: the vertex and attached edges
        * edge storage: byte array or hash

### Graph processing
* push-based: when computation ends, sends message
* pull-based: pull information from neighbors
    * graphlab: shared memory model

#### Consistency semantics
* push-based:
* pull-based: has race contention issues. needs locks over shared data
    * distributed locks, a vertex split across multiple nodes needs to sequentially locks over all local edges

#### Iterations
* synchronous: supersteps
* asynchronous: no need to wait for superstep to finish, could run eagerly
    * [connected component] (https://en.wikipedia.org/wiki/Connected_component_(graph_theory)) can not use synchronous, can not converge
    * [Greedy graph coloring] (https://en.wikipedia.org/wiki/Greedy_coloring), same, could not converge

### Other optimizations
* LALP: send only one message to one worker, the message would be replicated inside that worker
* master computer:
    * working/active vertices are small
    * master takes over sequential execution of remaining graph
    * master track global state
* dynamic migration of vertices: for balancing

### Conclusion
* giraph scales better with larger graphs, same number of machines
    * use byte array, more efficient
    * vertex mirroring causes significant network traffic
* graphlab scales better with larger number of machines for the same input graph
    * least contentions
* Why graphlab asynchronous is slow?
    * if the number of machines is small, it is fine
    * too many distributed locking