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
                * Schedule is **conflict serializable** if and only if the serialization graph has no cycles

#### [Serializability] (http://courses.cs.vt.edu/~cs5204/fall99/distributedDBMS/serial.html)
* Serialization graph
    * nodes for transactions
    * edge from T1 to T2 if there is a RW, WR, WW conflict from T1 to T2

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
* once a transaction releases a lock, it can **never** get another lock
    * Proglems: e.g., T1-W(X), T2-R(X), T1-abort, which could T2 read wrong data
        * workaround, cascading killing everyone touch X; T2 should not commit before T1; Strict 2PL

Theorem: 2 Phase Locking means only conflict serializable schedules result

#### Strict 2PL
Hold all locks untim commit or abort. Releasing phase only lasts for the last moment

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
    * locks during update, not the whole transaction
    * loss update: T1 lock update unlock, T2 lock commit unlock, T1 abort
        * there is not a problem if T1 commit rather than abort
        * in practical, too crazy: the transaction is unrecoverable, it commits before the end of transaction
* degree 1 long write locks
    * X locks on update items, hold it to the end
    * dirty read: T1 lock update unlock, T2 read directly and forcely, T1 abort (T2 breaks the rule: get a no-lock, T2 not even ask the lock manager)
* degree 2 long write, short read
    * to solve dirty read: T2 should try obtain a S lock before reading
    * unrepeatable read: T1 update X, commit; T2 read X; T3 update X; T3 commit; T2 read X
        * programmer could not avoid this problem by using temp variable
            * temp would be too big
            * implicit join performed by the DB itself
* degree 3 long write, long read == 2PL (strict 2PL)


##### Lock manager maintains db strcuture
* is a hashtable
* lock request: (object_id, transaction_id, lock)
* record: (rid, fileid, ...)
P is an object

1. imagine in a queue: T1 has S lock, T2 has IS; T3 wants X lock; T4 X; T5 S; T6 IS
- T3 wait (blocked) for T1 T2 finish, T4 T5 is waiting too
    * why not let T5 finish its S in a compatible group (executed along with T1 T2)?

lock and latch

* locks consists serializable capability of transaction actions
    * lock cannot guarantee serializbility (e.g., short write lock)
    * lock is on the higher level
* latch exclusive excess to physical data structures
    * exists within the data structure
    * more lightweight than locks (an external hashtable)
    * latch will prevent one page from moveing during a read (operation) by others

Suppose: T1 S, T2 S (there will come a T3 S later)

* if T1 wants upgrade to an X, how?
    * T1 would hang on S and want X (T1 would wait for T2 to finish)
* if T2 also wants to upgrade?
    * put T2 in the tail of queue
        * what if there is an T3 also has S?
            * if T1, T2 waits after T3 without releasing read lock, it would be deadlock
            * so, put T1, T2 upgrade requests before T3, while still hold their S locks
                * T1, T2 causes a deadlock (ask X while not release S) (T1 S, T2 S, T2 asks X, T1 asks X, T3)

#### Deadlock detection
build arcs for waiting relation (e.g., T1 watis for T2), search for cycles

deadlock resolution

* pick and transaction and kill: victim selection
    * youngest transaction

How do granularity decide to do

* lock deletion, start at record level, a page scan transaction
* if transaction locks >=3 records in a page, escalate to page lock
* if locks >= pages in a file, escalate to a file lock

### Optimisitc CC
* build serialization graph, validate: is anything unserializable
    * kill those transaction on overlapped data (please kill myself)

Procedure: three phases

1. Read: All writes are private
- Validate: Check for conflict
- Write: Make private write public

Consider, Ti, Tj, 3 condiitons ensuring it is consistent to assume Ti precede Tj

1. Ti completet write phase before Tj starts raed phase
- WriteSet(Ti) INTERSECT ReadSet(Tj) == empty and Ti completes write phase before Tj starts write phase
    * because Ti read < Ti write < Tj write
- WriteSet(Ti) INTERSECT ReadSet(Tj) == empty and WriteSet(Ti) INTERSECT WriteSet(Tj) == empty
and Ti completes read before Tj completes read

Question, who is Ti, who checks whom?: The later checks the previous one, I am always Tj

#### Serial Validation
first two scenario above could use serial validation

global transaction number, tnc:

* a "real" assignment of a transaction number takes place only after the write phase
* unneccesasary to assign transaction number to read-only transactions
* at the end of read phase, tnc is assigined to finish_tn

        << Critical Section Code:
        #: next transaction would have currentTransactionNumber, indicates who after who
        finish_tn = currentTransactionNumber
        valid = true
        # for all the transactions are running during my validation phase
        for t from start_tn + 1 to finish_tn do
            # there is WR conflict
            if WS(t) INTERSECT RS(this transaction) != empty:
                then valid = False
        if valid:
            then {write-phase, currentTN++, tn = currentTN}
        >>

#### Parallel Validation
would handle the third scenario

    << Critical Section Code:
    finish_TN = currentTN
    finish_active = copy of active
    #: transactions completed read phase but not write phase
    active = active UNION this transaction
    >>

    # This part is concurrent
    valid = True
    # these validated before me
    for t from start_tn to finish_tn do:
        if WS(t) INTERSECT RS(this transaction) != empty:
            then valid = False

    # these are validating at the same I am validating
    for t in finish_active:
        if WS(t) INTERSECT RS(this transaction) != empty or
            WS(t) INTERSECT WS(this transaction) != empty:
            then valid = False
    if valid then {
        write-phase();
        <<
            currentTN++
            tn = currentTN
            active = active - {this transaction}
        >>
    } else
    << active = active - {this transaction} >>

Note: write-phase() should be done before currentTN++ to prevent new transaction from skipping check this transaction which is still in write phase

1. Locking good in high-contention workloads
    * reason: with locking XACTs wait, rather than executing & aborting (optimistic would abort frequently, it is a waste of time)
- Low contention is good for optimistic
    * reason: but maybe not as good as hope, since cost of getting lock is not that much worse than maintaining RS & WS, RS & WS costs

### Timestamp CC
* each XACT gets a timestamp when it starts
* each data item X has
    * RTS(X): read timestamp of most recent read
    * WTS(X): write timestamp of most recent write

***
    if a XACT with timestamp T wants to read X:
        if T < WTS(X):
            T aborts
        else:
            read proceeds;
            set RTS(X) = max(RTS(X), T) # because there is younger read
    if XACT with timestamp T wants to write X:
        if T < RTS(X):
            T aborts
        if T < WTS(X):
            T aborts* # * can also just ignore write, XACT would proceed after it
        else:
            write proceeds;
            set WTS(X) = T

#### Multiversion CC
multiple version of data items

* RTS(Xi): read timestamp of transaction Xi
* WTS(Xi): write timestamp of transaction Xi

read of X by transaction with timestamp T

1. find version Xi with largest write timestamp <= T (most recent write timestamp)
- read it & set RTS(Xi) = max(RTS(Xi), T)

Write of X by transaction with timestamp T

1. find most recent version of Xi with WTS(Xi) < T

        if RTS(Xi) > T:
            abort
        else:
            # even if T < largest WTS(Xi)
            create new version Xj
            RTS(Xj) = WTS(Xj) = T

##### What if dirty read occurs? Read new version X, while it aborts afterwards
* New version are "private" until transaction commits, not let others see it
* At commit time, transaction gets a "commit-timestamp"
    * want to create new versions of updated objects with WTS = commit-timestamp (the WTS is postponed)
* if any other T2 with commit-timestamp in (start, commit) interval of this XACT, abort (first commit wins)
    * However, Oracle insists on first locker wins
* else commit, make updates visible

###### If the problem could be solved by NFA, then we can get DFA, then the algorithm

### A Critique of ANSI SQL Isolation Levels
ANSI phenomena

* P1(dirty read): T1 modifies X, T2 reads X, T1 aborts
* P2(non-repeatable reads): T1 reads X, T2 write X, T1 re-reads X
* P3(phantom): T1 reads data items satisfying predicate P; T2 inserts record satisfying P; T1 re-reads items satisfying P

Annotations

* w1[x] XACT 1 writes x
* r2[y] XACT 2 reads y
* c1 XACT 1 commits
* a1 XACT 1 aborts

#### Problems
    P* is wide interpretation, is not precise English, proposed by ANSI
    A* is strict interpretation
    A1: dirty read: w1[x] r2[x] (a1 and c2 in either order)
    P1: w1[x] r2[x] ((c1 or a1) and (c2 or a2) in either order)
    A2: r1[x] w2[x] c2 r1[x] c1
    P2: r1[x] w2[x] ((c1 or a1) and (c2 or a2) in either order)
    A3: r1[P] w2[y in P] c2 r1[P] c1
    P3: r1[P] w2[y in P] ((c1 or a1) and (c2 or a2) any order)
    P0: dirty write: w1[x] w2[x] ((c1 or a1) or (c2 or a2) in any order)

#### Isolation Levels
* Degree 0: no read locks, short write locks
* Degree 1: Locking Read Uncommited - no read locks, long write locks
* Degree 2: Locking Read Commited - short read locks, long write locks
* cursor never goes back
    * Cursor Stability: short read locks on current of cursor, short predicate read locks, long write locks
        * what's the difference from Degree 2's short read locks
* Locking repeatable read, long read locks, short predicate locks, long write locks
* Degree 3 = Locking serializable = long read locks on items & predicates, long write locks

Relations:

    Serializable A5B - write skew
    | P3 phantoms
    Repeatable read A5B A3
    | P2
    Cursor Stability
    | P4C (loss update) (ANSI miss this)
    Read Committed (Degree 2) - A5A - Snapshot Isolation
    | P1
    Read Uncommitted
    | P0
    Degree 0

#### Snapshot Isolation
A5A(Read Skew):
`r1[x] w2[x] w2[y] c2 r1[y] (c1 or a1): r1[y] reads the old snpashot version before XACT2`

A5B(Write Skew):
`r1[x] r2[y] w1[y] w2[x] (c1 and c2)
e.g., r1[x=50] r2[y=50] r2[x=50] r2[y=50] w1[y=-40] w2[x=-40] (c1) c2 (constraint x+y>0) This would happen since XACTs use the same version`

Snapshot isolation is multiversion CC, which prevents dirty reads/writes, A5A but not A5B. So READ COMMITTED << Snapshot Isolation

### B-Trees
Look for the right sibiling if the target key is greater than hi-key which stores the biggest value of this slot

Algorithm for modifying a page x:

    lock(x)
    A = get(x) // read x into memory
    modify data in A
    put(A, x) // write changes to disk
    unlock(x)

Search for v (always keeps an eye on the right hi-key):

    current = root
    A = get(current)
    while (current is not a leaf) {
        # "scannode" finds appropriate pointer in A when looking for v
        current = scannode(v, A)
        A = get(current)
    }
    while ((t = scannode(v, A)) is a linkpointer) {
        current = t
        A = get(current)
    }
    if v is in A:
        return SUCCESS
    else:
        return FALSE

Insert (lock chaining: require 2 locks at the same, throw one and get new one):

    Initialize stack
    current = root
    A = get(current)
    while (current is not a leaf) {
        t = current
        current = scannode(v, A)
        if (current is not a linkpointer)
            push t
    }
    lock(current)
    A = get(current)
    move_right()

DoInsertion:

    if A is safe: # A has room
        insert new key/ptr pair on A
        put(A, current)
        unlock(current)
    else: # has to split
        u = allocate new page for B
        redistribute A over A & B
        y = max value on A now
        make high key of B equal old high key of A # strech node A into two nodes, but the maximum remains the same
        make right link of B equal old right link to A
        make high key of A equal y
        make right link of A point to B
        put(B, u)
        put(A, current)

        # update the parent to prevent the horizontal link lists from becoming too big
        oldnode = current
        new key/pointer pair = (y, u)
        current = pop(stack) # picking the parent
        lock(current)
        A = get(current)
        move_right() // might hold 3 locks
        unlock(oldnode)
        goto DoInsertion

#### Problems
* Why needs to check the right sibiling after checking the hi-key?
    * if new data is bigger than hi-key, it should check if current node has not been splitted
* What if not requiring 2 locks before insert?
    * current node may be splitted by other XACT, then the new data may fail to find the right node to insert(not simply the right sibling)

### Summary
#### Locks
* Why non-strict 2PL is not equivalent to the four levels consistency
    * Level 0, 1 do not have read locks
    * Level 2 allows short read, which means you could acquire it later
    * Level 3 only releases write lock after commit
* Importan things about lock paths
    * writers have to lock all paths, while readers only have to lock 1 path
    * implicit locks should be considered
        * to get implicit X lock on a node, all of its parents must be locked in X mode
            * if the node is record, it would affect all the pages and indexes
        * if only lock in IX mode, another XACT may get IS then request S lock
* Does it make sense to use SIX on a record?
    * No, if record is the lowest granularity. Normal S or X would help
* When upgrades an SIX to X lock on a file, where should this upgrade request go in the requests queue?
    * This request should go immediately after current "granted group" which include the SIX and some IS locks

#### Multiversion CC
* When is it safe to delete an old version?
    * When there is no active transaction that could use it – that is, the timestamp of the oldest active transaction is greater than the timestamp of the oldest version of the record.
    * A version can be deleted when the oldest transaction has a newer version that it can read
* How can a reader reads without blocks?
    * readers would read version matches current start time which would be checked by CC
* Give an example allowed in multi-version CC but not in strict 2PL
    * S2PL release its write (exclusive) locks only after it has ended. read (shared) locks are released regularly during phase 2

#### Optimisitc CC
* Three validation conditions ensure safe. Ti has already commited
    1. Ti completed write before Tj starts read
    - Ti completed write before Tj starts write. And WS(Ti) has no overlap on RS(Tj)
    - Ti completed read before Tj completes read. And WS(Ti) has no overlap with RS(Tj) or WS(Tj)
* If Ti proceeds Tj, can TNC of Ti > Tj?
    * Not possible in serial validation
    * Possible in parrallel validation
        * validation phases are not serialized. Ti could enter validation before Tj and exit after Tj
        * So that Ti will get larger TNC
* Where parallel validation is better than serial validation
    * validation and write phases could be overlapped if their RS WS has no conflicts
* Compared with 2PL, what is the extra information needs by Optimistic CC
    * read, write sets of transactions
    * private writer buffers for updates
    * "finish acitve" sets for parallel validation
    * current transaction number

#### B-link tree
* How can updates go wrong with only one lock during move_right?
    * If holds one lock, XACT would holds no lock during "move_right". Another transaction could insert or split previous nodes.
* What if there is no lock when performing read?
    * XACT may wait for read lock. After a while, it wakes up and finds the target item is gone(splitted)
    * How to prevent above anomaly?
        * if a key moves, it only moves to the right
        * adding right link pointers and high keys to access right siblings
* Why B-link could achieve high concurrency?
    * The key is not simply stay in a subtree or a leaf. They are connected by right-links and indicated by high keys
* Since index pages are not shared or updated in an arbitrary way, is there any requirments on them?
    * On updating, the page is read into private storage atomically. The reads only see a completed write or not see them at all. Writes so that can not conflict
* When will get 3 locks?
    * one on a sibling in a just-split node, and two at the next level up as it looks for the proper insertion point
