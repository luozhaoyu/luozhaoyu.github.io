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
    R   id  name    type    box
        1   dane    county  (lowerleft, lr, ul, upperright)

store spatio data, how to find overlapping rectangles?

1. Approximate shapes by "bonding rectangles"
- Lookup for overlap is 2-steps:
    1. find overlapping bounding rectangles (boxes)
    - verify rectangles using true geometry (if it is trully overlapping)

#### B+ tree
* flat tree ensures fan out
* no order in 2 dimensional space (how to decide putting in)

#### R-tree
Let M be max entries that fit in a node, m <= M/2 be a specified minimum

1. every leaf node contains between m and M records (unless it is the root)
    * node splits if is more than M, each new node has M/2 records
- for each index record (I, tid) in a leaf, I is the smallest rectangle containing the object represented by the tuple
    * tid: tuple id, for one row
    * I is bounded box
- every non leaf node has between m and M children (unless it is the root)
- for each entry (I, tid) in an internal node, I is the smallest rectangle that contains all rectangles in its child
- the root has at least 2 children (unless it is a leaf)
- All leaves appear at the same level (it is not unbalanced, it has wide fan out)

##### Search: given an R-tree with root T, find all index records whose rectangles overlap a search rectangle S
How to find spatio data: search multiple path or replicate to multiple nodes (R* does), there is no way to avoid

1. if T is not a a leaf, check each entry E to see if E.I overlaps S. For all overlapping entries, search E.p
    * entry is (rectangle, pointer)
- if T is a leaf, check all entries to find overlaps

##### Insert E into R-tree
1. Invoke **ChooseLeaf** to select leaf L in which to place E
    1. why choose: since there are many choices to place an E, while B tree has only one place
    - set N to be root
    - if N is a leaf, return N
    - if N is not a leaf, let F be the entry whose rectangle F.I needs the least enlargement to include E.I
        * break ties by choosing smallest area rectangle
        * think about, why we prefer small rectangle?
            * not for fast access in one rectangle
            * for fast search of the whole index
- If L has room, install E. Otherwise, invoke **SplitNode** to obtain L and LL containing E and all the old entires of L
    * you have to decide which entries to L or LL in SplitNode, since there is no ordering
- Invoke **AdjustTree** in L, also passing in LL if a split occures
    * "fixes" bounding boxes after insertion, ensure its parents to enclose the expanded leaf


##### Splitting nodes
* Goal: minimize area of bounding rectangle for the 2 new nodes
* paper gives exponential(exact) quadratic linear(approximate) algorithm

### [Bit-map indexes] (http://en.wikipedia.org/wiki/Bitmap_index)
each index use 1 bit to represent the index

    B+ good at:
    Select S.name
    From Students S
    Where Sid=727,536

    has many distinct value, so B+ is good at
    but bad for bitmap, too many indexes

    B+ not good at:
    Select count(S.name)
    From Students S
    Where S.age=22 AND S.gender=M

    bitmap index is 1 if age=22, otherwise 0
    use & to combine indexes together, get 1 only

#### Bitmap index on column, say R.A
1. map from value to bitmap: there are many values and many corresponding bitmaps; we could use hashmap for mapping
    * one bitmap per value in R.A
    * bit in bitmap for value v is "1" in bit i if R.A has val "v" in ith tuple
- Suppose each page or R has K records on it (However, it would not work since we have variable K, not all number of records per page is the same)

        # file has to be sorted
        position i is:
        page: i div k
        record: i mod k

- What if variable number of records per page?
    * let k_max be max possible records per page
    * suppose file has n pages
    * then bitmaps have n*k_max entries
    * this solution inflates the indexes
        * this also need one additional bitmap for "phantom" tuples to distinguish if this bitmap corresponds to "real" tuple

#### bit-slice index

    R(A, B, C, Sales)
    a1,b1,c1   01100011
    a2,b2,c2   01110111
    a3,b3,c3   10110111

the bits are stored in a big binary blob, during accessing first slice, the program would loop all the words and find the first bit of each word. Bitmap slice could not access the bit directly

#### Join index

    RID emp(id, name, age, did)
    0x436 (7, "jane", 26, 3)
    build index on name maps from name to RID of emp record with that name

    RID dept(did, dname, ...)
    0x799 (3, "CS", ...)

    "regular" index:
        entry: ("jane", 0x436)

    However, we could also build index map from emp.name to dept.name
    join index:
        entry: ("jane", 0x799) # RID is from a different table; the index is precomputing the join

#### [User-defined function] (http://en.wikipedia.org/wiki/User-defined_function)
* select R.A from R where **f**(R.B) >= 10
    * f is UDF: such as convert concurrency
    * f must be deterministic
* problem is query optimizer could not make an estimate
    * PL people are not afraid to analyse your JAVA codes

What to do with bitmap index?

* can build an index on a function result, i.e., build index on **f**(R.B)
    * build a table with (RID, f(R.B))

* R(A,B,C,D,E,blob) build index on R(A,B,C)
    * maps from (a,b,c) tuples to RIDS: (a1b1c1, 0x799)
    * a "covering index" Q: means can answer Q from index alone
* think `Q: select R.A, R.B from R`: can use index to answer
    * rather than scanning the table, scan the smaller index

### Column-store
store columns instead of rows, scan attribute would be much faster



### Summary
#### Query Optimization
* Why does the System R optimizer use the “only left deep trees” rule?
    * given the join algorithms System-R had at the time, left-deep trees can be pipelined but right-deep trees cannot.
        * we could start scan once there is joined data popped upper, streaming
* What are two statistical assumptions that System R makes in doing selectivity estimation?
* Query optimizers make use of estimates of cardinalities when estimating the cost of plans. Suppose you were able to magically “reach in” and convert some of the cardinality estimates to their true, exact values. Would this necessarily result in better query plans? Why or why not?
    * No. We should use estimates compare with estimates to get relative different. Since the estimations may have systematic error or consistent skew, we may get wrong comparision between a real value with estimate value
* Why is it important to keep track of “interesting orders” instead of just calculating the single best one-table plan for each table in the query?
    * single best one-table plan is a greedy optimization algorithm which may fail to achieve global best
    * global best may consist of slower plans
* The query optimizer might not eliminate a suboptimal sub-plan if it generates an “interesting order.” What constitutes an “interesting order” and why are plans that generate them retained?

#### R-tree
* what is the goal of SplitNode algorithm?
* Why you might have to search multiple paths during a lookup in an R-tree. Give an example. Why not in B-trees
* Why trying to minimize the resulting area of two new nodes in SplitNode of R-tree
* Why store rectangle not precise polygon?
    * the rectangle is easy for computing

#### Bitmap
* `SELECT * FROM R WHERE R.a=c1 and R.b=c2` whether use bitmap or B-tree indices on R.a and R.b: use RIDSize, RIDCompTime, WordLength, BitwiseAndTime, NumAValues, NumBValues in your equations
* Consider a bitmap index built on column B of a table R(A, B, C). Suppose
R has 1,000,000 rows in it, that the minimal size of an R tuple is 100 bytes, and that
data pages can hold 10000 bytes of user data. Furthermore, suppose that R currently
is stored on 13,000 pages (that is, tuples of R appear on 13,000 pages) and R.B has
200 distinct values. Finally, also assume that the method described in class for
mapping bit position to record in page is being used.
    * How many bitmaps will be in the index?
    * How many bits will be in each bitmap?
    * What is the total number of “1” bits in all of the bitmaps?
