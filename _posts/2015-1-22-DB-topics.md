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
lock deletion, start at record level, a page scan transaction
if transaction locks 3 records in a page, escalate to page lock
pages in the file escalate to a file lock

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

    < Critical Section Code:
    #: next transaction would have currentTransactionNumber, indicates who after who
    finish_tn = currentTransactionNumber
    valid = true
    # for all the transactions are running during my validation phase
    for t from start_tn + 1 to finish_tn do
        # there is WR conflict
        if WS(t) INTERSECT RS(this transaction) != empty:
            then valid = False
    if valid:
        then {write phase, currentTN++, tn = currentTN}
    >

#### Parallel Validation
would handle the third scenario

    < Critical Section Code:
    finish_TN = currentTN
    finish_active = copy of active
    #: transactions completed read phase but not write phase
    active = active UNION this transaction
    >

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
        <
            currentTN++
            tn = currentTN
            active = active - {this transaction}
        >
    } else
    < active = active - {this transaction} >

