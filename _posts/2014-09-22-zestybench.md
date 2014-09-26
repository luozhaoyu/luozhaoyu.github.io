---
layout: post
title: "Zestybench, measure IPC"
---
This post is a digest of the first paper in my CS736 which will talk about the IPC benchmark briefly.

Source code in [available] (https://github.com/luozhaoyu/zestybench)

### Abstract

### Body
#### environment description
* Intel 4 core 3.2G CPU, L1 cache 32K, L2 cache 256K, L3 cache 6144K
* buffer size:
    * default read/write buffer size, net.core.rmem_max=122K net.core.wmem_max=122K
    * maxium read/write buffer size, net.core.rmem_max=122K net.core.rmem_max=122K
    * tcp threshold, net.ipv4.tcp_mem = 1521600 2028800 3043200, data > second value would let kernel buffer as much as possible, data > thrid value would cause packets loss
    * tcp read/write buffer size, net.ipv4.tcp_rmem = 4096 87380 4194304(unit is page size, which is usually 4KB), net.ipv4.tcp_wmem = 4096 16384 4194304
        - first value is the minimum buffer for each TCP **connection**
        - second value is default buffer for each TCP **socket** and **overrides** net.core.[r|w]mem_default
        - thrid value is the maxium buffer and **is overridden** by net.core.[r|w]mem_max.
    * udp thresholds, net.ipv4.udp_mem = 1521600 2028800 3043200
    * udp minimal receive/send buffer size, net.ipv4.udp_[r|w]mem_min = 4096
* bandwidth: 100MB/s, regulated by upper switch
* memory: 16GB in total, more than 5G free
#### clock precision
* clock_getres(), finds the resolution
* clock_gettime(MONOTONIC), even though REALTIME is fast

#### trivial kernel call
* getpid(), getuid() is simple enough

#### IPC
##### pipe
* first pipe(), then fork()

##### AF_INET tcp
* TCP_NODELAY, disable the [Nagle algorithm] (http://en.wikipedia.org/wiki/Nagle's_algorithm), because of the need of real time responses
* server is using epoll_wait
* system default tcp buffer is enough

###### latency
* server echos the received data immediately

###### throughput
* server only ack the number of received data immediately

##### AF_INET udp
* System default udp buffer is 122K, which should be enough. But packets may still get lost over the ethernet. Splitted packets would become worse
* server is using epoll_wait
* client could adjust its send buffer size

###### latency
* ensure the success delivery of data, client send X data, server received Y data and reply, client would receive Z data in the end. loop until SUM(Z) >= total data

###### throughput
* regardless of receiving, client just throw piles of data to the server directly, and then read all the responds
    * client should not use block reading in this situation, since the responds is unpredictable. So client will epoll_wait for 10milliseconds before its exit and the last respond time will be recorded

##### AF_UNIX tcp
##### AF_UNIX udp


### reference
* [IP options] (https://www.frozentux.net/ipsysctl-tutorial/chunkyhtml/)
