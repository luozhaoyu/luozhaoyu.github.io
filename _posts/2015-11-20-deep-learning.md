---
layout: post
title: "Project Adam: Building and Efficient and Scalable Deep Learning Training System"
---

### Why Deep Learning is hard?
* extremely hard to construct appropriate features

### Why Adam?
* Optimizing and balancing both computation and communication for this application through whole system co-design
    * partition large models acroos machines
* Tolerate inconsistencies well
    * multi-threaded model parameter updates without locks
    * asynchronous batched parameter updates (**asynchronous training improvs model accuracy**)
* faster and more accuracy

### Adam
* Adam system is general-purpose
    * since stochastic gradient descent is a generic training algorithm that can train any DNN via back-propagation

#### Components
* A global parameter server: all model replicas share a common set of parameters that is stored here
    * Throughput Optimization: model parameters are divided into 1 MB sized shards
    * Delayed Persistence: Parameter storage is modelled as a write back cache, with dirty chunks flushed asynchronously in the background (agin since NN would converge)
    * Fault Tolerant Operation
        * primary use 2PC to replicate
* Model replicas:
    * operates in parallel
    * asynchronous publishes model weight updates to and receives updated parameter weights from parameter server
        * NN is itself resilient learning architecture, so that stale parameter would not be a problem
* Parameter server controller machines: a Paxos cluster
    * hand out bucket assignments to parameter servers and persists the lease information in its replicated state
    * if primary is lost, they would elect one of the secondary and assigns a new lease for this bucket

#### Model Training
* partition models vertically across model worker machines to minimize the amount of cross-machine communication that is required for the convolution layers
* Multi-Threaded Training
    * Each thread allocates a training context for feed-forward evaluation and back propagation
    * Both the context and per-thread scratch buffer for intermediate results use NUMA-aware allocations to reduce crosff-memory bus traffic
* Fast Weight Updates
    * access and update the shared model weights **locally without using locks**
    * since NN are resilient and weight updates are associative and commutative which lead to convergence
* Reducing Memory Copies: pass a pointer to the relevant block of neurons
* Memory System Optimizations
    * partition models across multiple machines such that the working sets for the model layers fit in the L3 cache
    * cache locality: prefer a row major or column major layout for the layer weight matrix
* Mitigating the Impact of Slow Machines
    * allow threads to process multiple images in parallel
    * waiting for 75% of model replicas to complete processing (which is accurate enough emprically)
* Two different parameter server communication protocols:
    1. locally computes and accumulates the weight updates in a buffer that is periodically sent to the parameter server machines when k (~100) images have been processed
    - send the activation and error gradient vectors to the parameter server machines for the fully connected layeres
        * since parameter servers are CPU underutilization