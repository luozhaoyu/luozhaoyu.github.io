---
layout: post
title: "Google Datacenter Network"
---

### Network Construction Principles
* [Clos topologies] (https://en.wikipedia.org/wiki/Clos_network)
    * could scale to arbitrary size
    * in-built path diversity and redundancy
    * problems: managing fiber fanout, complex routing across multiple equal-cost paths
* Merchant silicon
    * trade-offs
        * failures are more frequent
        * small shared buffer
        * ECMP(equal cost multi path), network balancing (lots of contention)
* Centralized control protocols
    * each switch calculate forwarding table from a dynamically-elected central switch
    * central control and configuration (SDN)

### Background
* traffic doubles every year
* high bandwidth applications had to fit under a single Top of Rack to avoid heavy oversubscribe uplinks
* maximum cluster scale
    * small cluster may fail to hold one large job
    * would increase availability, ensure power diversity and lower storage overhead (think about a cluster failure)
* rack and power diversity eliminates locality and drives the need for uniform bandwidth across the cluster
    * so that storage placement and job scheduling have little locality

### Evolution
* traditional 4 post cluster: 4 cluster 512 * 1G, 512 racks, 40 servers/rack
    * oversubscription is 10
    * schedule for locality
    * keep storage also local
* FH1.0: the ToR switch radix is too small
    * blocking means could not use full speed at a time, there is contention
* FH1.1
    * buddy two ToR switches together
        * could use two switches for bursting, bypass through the connected switch
    * The stage 2 and 3 switches within an aggregation block were cabled in a single block (vs. 2 disjoint blocks in FH1.0) in a configuration similar to a Flat Neighborhood Network
    * put FH1.1 and old Four-post cluster routers together, FH1.1 only handles intra-cluster traffic
    * *drawback*: external copper cabling
* Watchtower
    * use merchant silicon switch, 16 * 10G, to build a traditional switch chassis with a backplane
    * Fiber bundling redueced the cabling challenge and fiber cost
    * depopulated deployment, 50% capacity initially deployed
* Saturn: further increase maximum cluster scale
* Junpiter
    * expanding Clos fabric across the entire datacenter subsuming the inter-cluster networking layer
    * incremental deployment


### Features
* WCC: Decommisiioning Cluster Routers
    * build a seperate aggregation block for external connectivity, which is more easy to controll the effect
* Inter-cluster connections
    * Freedome block
* Build own control plane
    * need support for multipath, equal-cost forwarding
    * need network manageability
* Routing
    * switches learn link state through pair-wise neighbor discovery
    * switches exchange their local view of connectivity with Firepath master
        * it is eventual view consistency
    * implement a Firepath master election protocol
* Configuration and Management
    * distribute a single monolithic cluster configuration to all switches
        * each switch may have update new config
    * extened ICMP, random traceroutes probes
    * static config of cluster topology
    * deltas of topology changes (only track neighbours)
* [ECMP] (https://en.wikipedia.org/wiki/Equal-cost_multi-path_routing)
    * central scheduling of long flows, splitting flows in near uniform sized segments at servers ([TSO] (https://en.wikipedia.org/wiki/TCP_segmentation_offloading))
        * all flows are the same size
        * [elephant flow problem] (https://en.wikipedia.org/wiki/Elephant_flow)
        * bandwidth * delay
* Why [BGP] (https://en.wikipedia.org/wiki/Border_Gateway_Protocol)
    * if one fails, use another AS number would be easy (switch quickly)
    * topology is symmetric, 2 level, simple path vector updates, one localized -> converge quickly
    * OS implementation of BGP is good

#### Fabric Congestion
* high congestion drops as utilization approached 25%
* QoS
* tune host to bound TCP congestion window for intra-cluster traffic to not overun the small buffers in switch chips
* link-level pause to keep servers from over-running oversubscribed uplinks
* ECN
* monitor application bandwidth in the face of oversubscription ratio
* dynamically allocate a disproportionate chip buffer to absorb temporary traffic burst
* good switch hash function to ensure load balancing

#### Outages
* control software problems at scale
    * liveness protocol and route computation contented for limited CPU, which delayed response to heartbeat message
    * test control software in virtualized environment
* aging hardware with unhandled failure modes
    * one should monitor fabric backplane and CPN (Control Plane Network) links
* misconfiguration of certain component
    * need locking mechanisms to help simultaneous read/write access to BGP configuration

### Comments
* Layered network is a good methodology to achieve large network capacity and ensure availability
* The network devices themselves in large network would also meet the distributed system problems
    * how to elect central master
    * how to keep configuration consistency
    * how to implement transaction in wide range updates
    * autonomous switches design is more complex and hard to implement when compares with centralized control
* Incremental deployment, depopulate deployment are all aim at eliminate the upgrading risk, and they are good feature for network robustness
* Large scale LAN needs its own customized protocol to handle network failure or congestion
* Seldom company or organization has the power to design its own switch
* It is interesting to find out the granularity of power hierarchy is row, so larger cluster would have better availabity and lower need for storage redundancy
    * Cluster becomes an autonomous computing unit
