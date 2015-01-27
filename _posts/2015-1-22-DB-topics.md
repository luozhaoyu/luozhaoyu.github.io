---
layout: post
title: "Topics in Database Management Systems"
---

### Concurrency Control
* much more deeper detail
* efficiency algorithm (performance rather than correctness)
* degree of consistency (between chaos and perfect)
* What concurrent access to data?
    * performance (e.g. transaction:
        * T1: transfer moeny from A to B
        * T2: giving 6% interest rate to all accounts)
        * Idea: structure access to DB as a bunch of **Transactions**
            * each transaction takes DB from consistant state to consistant state
            * then serial execution of a set of transactions guarantees consistency
                * want to run transactions concurrently but it should "look like"  they ran serially
            * certain "conflicts" enforce another on the transactions
                * WR, RW, WW
                * Techniques: build serialization graph
                    * nodes for transactions
                    * edge from T1 to T2 if there is a RW, WR, WW conflict from T1 to T2
                * Schedule is **conflict serializable** if and only if the serialization graph has no cycles

### 2 Phase Locking
* S: Shared (read anything without any **more** locks)
* X: eXclusive
* IS: Intension Shared (have the permission to **ask** for S **later**)
* IX: Intension eXclusive
* SIX: S + IX, S on the file, IX can lock file later and lock record X
* S conflicts with IX: S holds the read, IX will never acquire X later
* IS would not conflicts with IX: they will deal the lock later

    Compatibility:
        NL  IS  IX  S   SIX X
    NL  Y   Y   Y   Y   Y   Y
    IS  Y   Y   Y   Y   Y   N
    IX  Y   Y   Y   N   N   N
    S   Y   Y   N   Y   N   N
    SIX Y   Y   N   N   N   N
    X   Y   N   N   N   N   N


* before reading a data item, get on S-lock on it
* before writing, get an X-lock (like read-write lock)
* once a transaction releases a lock, it can never get another lock

Theorem: 2 Phase Locking means only conflict serializable schedules result

### Granularity of Locks
#### granularity
* T1: update balance of account 517,231
* T2: compute sum of all IM balances
* T3: scan university payroll, lower salary of all TAs making over $100,000
    * read many, update few

Strategies

* Records Lock
    * great for T1 (high concurrency)
    * bad for T2 (too many locks)
* Files Lock
    * great for T2 (low overhead)
    * bad for T1 (low concurrency)
* T1 locks at record level; T2 locks at file level
    * idea1: set intention lock on path to the object you want to lock (lock file for T1 record lock)
        * protocol: to get an S(X) ock on an object, get IS(IX) on all ancestors in the granularity hierarchy
            * DB(IX)->File(IX)->Page(IX)->Record(X)
    * idea2: get locks top-down, release bottom up
    * exclusive access lock all paths; shared access lock one
        * path: DB->Index->Record
        * need to consider implicit & explicit locks
            * explicit lock page means implicit lock records, which implicit lock all the indexes
                * However, lock indexes is very weired (it would block everyone out) (correct, but inefficient)
    * writer get IX locks on all paths to object they wish to X-lock

#### consistency
Lock all table while scanning cause concurrency problem, see
[A Critique of ANSI SQL Isolation Levels] (http://pages.cs.wisc.edu/~cs764-1/critique.pdf)

* degree 0 short write locks
* degree 1 long write locks
* degree 2 long write, short read
* degree 3 long write, long read == 2PL (strict 2PL)
