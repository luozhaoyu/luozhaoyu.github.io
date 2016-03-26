---
layout: post
title: "Sparrow Omega"
categories: blog
---

### Sparrow
* Yarn: centralized scheduling: limited throughput
* scheduling without coordination
    * sparrow distributed scheduling
    * omega uncoordinated, multi-schedules

#### Sparrow scheduling unit
* schedule "m" tasks at a time
* 1st level: constrained
    * data locality is largely constrained
* unconstrained

#### Sparrow scheduling goals/non-goals
* Goals:
    * high-fault tolerance (scheduler)
    * very high throughput/ low latency
    * throughput scales with jobs
    * as simple as possible
* Non-goals:
    * explicit job-level fairness guarantees
    * some constraints
    * bin-packing

#### mechanism
* one query: multiple jobs
    * one job: multiple stage
        * scheduling same stage at once
* one scheduler per job
    * batch-sampling
* each executor is a JVM
* each executor has a task queue
* per-task sampling
    * sampling maybe highly random, may choose two slow executors
    * **problems** in probing some executors
        1. race conditions
        - their queue length may not reflect the real condition

##### late binding
1. enqueue some reservation in the executor
- when executor encounter the reservation, it will ask for a task
    * schedule in m fastest responoding executors

constraints

* data locality -> per-task sampling on relevant nodes


##### Failure
1. one scheduler per "front end" instance (a query)
    * Each query can have other Front-Ends as backup
    * Backup can pick when primary fails, but apps decides if rescheduling is needed

#### Reference
* [Sparrow](http://www.cnblogs.com/fxjwind/p/3518871.html)

### Omega
* monolithic
* schedulers for different sub-cluster
    * leads to inefficiency
* multi-level scheduler (Mesos)

contentions

* when contention on same resource:
    * precedence across workloads, satisfy precedence firstly
    * optimistic concurrency control

