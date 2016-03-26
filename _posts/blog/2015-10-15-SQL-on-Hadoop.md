---
layout: post
title: "SQL-on-Hadoop"
categories: blog
---

### [SQL-on-Hadoop](http://www.slideshare.net/abadid/sqlonhadoop-tutorial)
* Both Hive-MR and Hive-Tez are CPU-bound during scan operations, which is Java deserialization
    * A faster Java deserialization lib or JSON lib would help a lot
#### Hive
* Tez eliminates the startup and materialization overheads of MapReduce
    * However, it does not avoid Java deserialization overheads during scan operations
* Tez pipelines data through execution stages instead of creating temporary intermediate files

##### Optimizations
* optimization of correlated queries
* predicate push-down and index filtering in ORC file
* map-side join and aggregation

#### Impala
* Impala is a shared-nothing parallel DB architecture
* `impalad` is long running daemons on every DataNode
    * it reduces job initialization and scheduling multiple tasks per node
* single threaded execution of joins and aggregations impedes performance
* Impala could not recover from mid-query failures
    * it needs to re-run the whole query
* without code generation, the query becomes CPU-bound

### TPC-H
* I/O is not Impala's bottleneck in TPC-H, but Hive needs ORC
* During scan operation, both Hive-MR and Hive-Tez are CPU-bound
    * because of Java deserialization
* Hive knows how to share a common input, but Impala does not, it scans table twice (but it is not bad, since they are cached)

#### Q1
* Hive procedure
    * scan `lineitem`
        - apply predicates
        - select columns
        - aggregate
    * map: do partial aggregate
        - shuffle to reduce
        - reduce: do global aggregate
        - sort
    * the number of mappers depends on the number of file partitions. e.g., the default replica is 3, at least 3 mappers
    * the number of reducers depends on the number of unique keys. It is <= number of distinct keys
* Imapla
    * I/O initial
    * no intermediate writes
    * no scheduling
    * single thread join and aggregation

#### Q17
* Hive
    * Hive map tasks:
        * read `lineitem`
        * do partial aggregate
        * read past-table
    * Hive reduce tasks:
        * do global aggregate
        * do join
        * do partial aggregate for the future join
* Imapala stages:
    1. scan past table, broadcasts to all nodes
    - read `lineitem`, do partial aggregate, shuffle partial results
    - read `lineitem`, join with past result
