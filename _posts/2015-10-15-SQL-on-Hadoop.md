---
layout: post
title: "SQL-on-Hadoop"
---

### [SQL-on-Hadoop] (http://www.slideshare.net/abadid/sqlonhadoop-tutorial)
#### Q1
* Hive procedure
    1. scan lineitem
    - apply predicates
    - select columns
    - aggregate
	1. map: do partial aggregate
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
* Hive map tasks:
    * read line item
    * do partial aggregate
    * read past-table
* Hive reduce tasks:
    * do global aggregate
    * do join
    * do partial aggregate for the future join

* Imapala stages:
    1. scan past table, broadcasts to all nodes
    - read lineitem, do partial aggregate, shuffle partial results
    - read lineitem, join with past result