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

Architecture

    RDS (sort merge join, aggregate)
    RSI-----------------------------(research storage interface)
    RSS (Research Storage System, manages buffer pools index scan)

##### SARGable
Definition:

* A SARGable predicate is one that looks like "attributue op value"
    * Push to buffer pools (buffer management) (RSS): R.a > 7, S.b = 17
    * NOT: R.a = S.b
* A SARG is an expression SARG1 or SARG2 or ... or SARGn (each of these is a conjunction of SARGable predicates)
* a predicate matches an index when:

        index on (A, B, C)
        matches:
            A = 7
            AB = (7, "fred")
        not matches:
            B = "jane" # B is not the prefix initial substring

    * predicates a SARGable
    * columns in predicate are initial substring of index key
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

- do:

        for each relation:
            calculate the cost of scanning relation
            for each applicable index & segment scan

- what is produced?

        case cost
        unique index matching predicate    1 + 1 + w # 1 for b-tree search, 1 for buffer pool, RSI call is 1
        clustered index I matching one or more boolean factors F(preds) * (NINDX(I) + TCARD(T)) + W * RSICARD
        # W * RSICARD: product of selectivities for all SARGable preds
        nonclustered index I

    1. cost C in form (number of pages fetches) + W * RSICARD
    - ordering of tuples produced (if any)
- find best single-table access plans (for each interesting order)
- find best way to join pairs from previous step (for each interesting order)
- find best way to join in one more table with result of previous step

The import notion is that: you should have your formula to do estimation

    Cost_of_nested_loop_join = Cost_of_outer(Path1) + Number_of_R_rows * Cost_of_inner(Path2)
    Cost_of_merge_sort_join = Cost_of_outer(Path1) + N * Cost_of_inner(Path2)
    # However, it is different from nested loop: Cost_of_inner is cheap due to sorting

##### Histograms
think about the age distribution in uw-madison, someone store age: SELECT(R.a == c)(R), unevenly distributed

people starts to store more statistic information as **histograms**

* equi-width: count for each age bucket (10, 20), (20, 30)
* equi-depth: each bucket has the same number (15, 17), (17, 18), (18, 19), (24, 28), (50, 100)
    * in practise, build it by sampling

"end-biased": equi-depth + exact count of k most frequent values

think about joining R.c == S.d:

1. join their histogram!
- 2-dimensional histogram, vertical is R, horizontal is S (pre-join)
    * `|R join S| = |R|`

##### Sampling
take sample of DB, run query over the sample, and measure the selectivity: too slow, if run every time

2 cases, think about their differences:

1. R.a is odd number from 1 to n, S.b is odd number from 1 to n (result it 0)
- R.a is odd number, S.b is even number (half join half)

Histogram do not know, sampling know it immediately

But it is still a difficult problem: `select distinct R.A from R`

##### Optimizer estimate

    R join S
                            IndexNestedLoop        NL
    10,000  ~   10s INL     100units    5000
    100,000  ~   10s INL    1500units   1,000,000
    1,000,000  ~   2yrs NL              -1087201831

Current thoughts:

* re-optimization
    * execute for a while (sampling), then re-involve optimization
* remember previous results (use it), but not work very well:
    1. only learn about things you executed: previous may be mistake
    - old approach: compare estimates to estimate; new approach: compare estimates to true values
        * new approach is worse
* robust query optimization: choose reliable plan than risky plan


#### Join trees (only use left deep trees)
right deep tree for data warehouse: assume we would join ABCDF, put hash table of ABCD in memory, then pipeline the F up

#### Schema
    F(A,B,C,Measure)
    F(storeId, date, product, quantity)

[Star schema] (http://en.wikipedia.org/wiki/Star_schema) (build data warehouse)

* dimensional table contain only [hierarchies] (http://en.wikipedia.org/wiki/Hierarchical_database_model)
* Fat table contains all the details

[Snowflake schema] (http://en.wikipedia.org/wiki/Snowflake_schema)

Cross product:

* System R avoids cross product if possible: if R join S join T, only R.a = S.b and S.c = T.d, not R.x = T.y (no cross product)
* if we join A B C D F:
    1. A join F
    - join B
    - join C
    - join D
* You could also do cross product by adding a dummy attribute to each tuple in each table

### ADT
user defined rich data type

### R-trees
store spatio data


### Summary
#### Query Optimization
* Why does the System R optimizer use the “only left deep trees” rule?
    * given the join algorithms System-R had at the time, left-deep trees can be pipelined but right-deep trees cannot.
        * we could start scan once there is joined data popped upper, streaming
* What are two statistical assumptions that System R makes in doing selectivity estimation?
* Query optimizers make use of estimates of cardinalities when estimating the cost of plans. Suppose you were able to magically “reach in” and convert some of the cardinality estimates to their true, exact values. Would this necessarily result in better query plans? Why or why not?
* Why is it important to keep track of “interesting orders” instead of just calculating the single best one-table plan for each table in the query?
* The query optimizer might not eliminate a suboptimal sub-plan if it generates an “interesting order.” What constitutes an “interesting order” and why are plans that generate them retained?
