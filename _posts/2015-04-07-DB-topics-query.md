---
layout: post
title: "DB topics - Query"
---

### Query optimization
Naive thinks and problems

1. enumerate all ways to evaluate the query (too many!)
- pick the fastest one (you do not know how fast the plans are)

How to test optimizer?

1. make it not worse
- big customers strategy, hack

#### System R
Main idea is to choose the best strategies (by scoring them ) seperately and combines them together

Access paths

* segments scan (file scan)
* index scan (B+ tree)
    * clustered index
    * unclustered index

Join methods

* Nested loops
* Sort merge

##### SARGable
Definition:

* A SARGable predicate is one that looks like "attributue op value"
    * Push to buffer pools (buffer management) (RSS): R.a > 7, S.b = 17
    * NOT: R.a = S.b
* A SARG is an expression SARG1 or SARG2 or ... or SARGn (each of these is a conjunction of SARGable predicates)
* a predicate matches an index when:
    * predicates a SARGable
    * columns in predicate are initial substring of index key

        index on (A, B, C)
        matches:
            A = 7
            AB = (7, "fred")
        not matches:
            B = "jane" # B is not the prefix initial substring
* an ordering of tuples is "interesting" if it can be used by GroupBy, OrderBy or Join

#### Cost of a query
Cost of a query: I/O times + Weighting factor * CPU time =
    number of page fetches (number of I/Os actually) + W * (number of RSI calls)

* I/O time:
* CPU time:
    * number of RSI calls: number of rows brought from RSS into ROS
* W: weighting factor
    * the faster the CPU, the better (could approximate 0)

#### Statistics
    Used to optimize, measure

    NCARD(T) = number of rows in T
    TCARD(T) = number of pages containing tuples from T
    P(T) = TCARD(T) / number of non-empty pages: in segment
    ICARD(I) = number of distinct keys in index I
    NINDX(I) = number of pages in index I

#### Cost calculation
1. calculate **selectivity factor F** for each boolean factor in predicate list

        Predicate           Selectivity estimate
        attr = value        F = 1/ICARD(I), if index exists (assume uniformity of a distribution among the index keys)
        attr = value        F = 1/10, if no index exists (assume uniformity of a distribution among the index keys)
        v1 <= attr <= v2    F = (v2 - v1) / (high_key - low_key), if index exists
        v1 <= attr <= v2    F = 1/4, if no index exists
        expr1 or expr2      F = F(expr1) + F(expr2) - F(expr1) * F(expr2)
        expr1 and expr2     F = F(expr1) * F(expr2) (assume independence of column values)




### Summary
