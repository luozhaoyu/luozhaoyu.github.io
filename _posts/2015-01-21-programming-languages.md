---
layout: post
title: "Principles of Programming Languages"
---

### Lambda Calculus
* Full-beta reduction (random)
* Normal order (leftmost/outermost)
* call-by-value, most programming language, evaluate arguments before passing them, evaluate expression inside parenthesis first
* call-by-name, no reductions inside abstractions (lazy strategy, call-by-need)

        Different order, different results:
        W=(lambda x.x x x)
        F=(lambda x.lambda y.y)
        I=lambdax,x

        F(WW)I
        -> F(WWW)I
        -> F(WWWW)I
        -> F(WWWWW)I

        ALTERNATIVE:
        F(WW)I
        -> (lambda y.y)I
        -> I

#### Precise definitions
* Scoping: which enclosing lambda a variable refers to ?
* Bound: a variable Bound if it is associated with lambda
* Free: not Bound

#### Structural definition
1. variable x -> x is free -> there are no bound variables
    * x (lambda x.x)
- lambda x.M -> every x is bound in M -> for all other variables y != x if y is free in M, then it is free in lambda x.M
    * M could be lambda y.y
    * FreeVariable(x) = {x}
    * FreeVariable(lambda x.M) = FreeVariable(M) - {x}
- Expression M N, x is free
    * FreeVariable(M N) = FreeVariable(M) U FreeVariable(N)

        lambda x.f x ((lambda x.y x) v) l
        -alpha-> lambda x. f x ((lambda z.y z) v) l
        -beta-> f l ((lambda z.y z) v)

variables can get captured

    ((lambda x.lambda y.x)w)z = (lambda y.w)z = w
    ((lambda x.lambda y.x)y)z != (lambda y.y)z
    -alpha-> ((lambda x.lambda d.x)y)z

#### Substitution
The substitution of a term N for all free occurrence of x in M. Do M[N/x]

1. x [N/x]= N
- y [N/x]= y iff y!=x
- M P [N/x]= M[N/x] P[N/x]
- (lambda x.M) [N/x]= lambda x.M
- (lambda y.M) [N/x]= lambda z.((M[z/y])[N/x]) where **y!=x** and **z is free**

Summary:
1. alpha reduction (renaming): lambda x.M -> lambda y.M[y/x] (y is fresh)
- beta reduction: (lambda x.M)N -> M[N/x]

#### Mulitiple arguments
* tru, fls, pairs, fst, snd
* church numerals
* operators:
    * Addition
        * \m.\n.\f.\x.m f (n f x)
        * \m.\n.n succ m: apply succ n times on m
    * Multiply
        * \m.\n.m (plus n) 0
        * \m.\n.m (n f)
* isZero: lambda m.m (lambda x.fls) tru


### Fiexed-point Combinators
OR = how to do recursion without recursion

    fact = \x.if (isZero x) then 1 else x*f(x-1)
        \f.\x.if (isZero x) then 1 else x*(ff)(x-1)
    fact` = fact fact
    U = \x.x x

We want a magic function Y such that: Y(F) -> T   F(T) = T

    Solve Y (what do we know?)
    Y(F) = F(Y(F))
    Y = \F.F(Y(F))
    Y = \F.F(\x.(Y(F)(x))) # just insert a variable

    Y = U(\h.\F.F(\x.((hh)(F)(x))))
    \f.(\x.f(\y.x x y))
        (\x.f(\y.x x y))

### The Church-Rosser Theorem
* proof of coloaries
* proof of Church-Rosser
    * import about walks, walk is similar to call-by-value
        1. walk can not be got by concatenate two walks
        - Two walks that reduce non-overlapping redexes can be concatenated to form a walk

### [SAT] (http://en.wikipedia.org/wiki/Boolean_satisfiability_problem)
* [CNF] (http://en.wikipedia.org/wiki/Conjunctive_normal_form)

#### [DPLL] (http://en.wikipedia.org/wiki/DPLL_algorithm)
* [DPLL algorithm, Pure literal propagation] (http://www.dis.uniroma1.it/~liberato/ar/dpll/dpll.html)
* Pure literal propagation
    * if p appear only positively in all clauses, then set p to true
    * if p only appears !p set it to false
* [Resolution] (http://en.wikipedia.org/wiki/Resolution_(logic))
* [Boolean Constraint propagation] (http://en.wikipedia.org/wiki/Unit_propagation)
    1. Decision variable: variable assigned in Decide Step
    - decision level: the level/order in which variable assgined
* [Unique implication point] (http://www.cs.princeton.edu/courses/archive/fall13/cos402/readings/SAT_learning_clauses.pdf)
* non-chronological backtracking: backtrack to level d where conflict clause C is an asserting clause

Implication graph: lead to a conflict, subgraph characterizes the conflicts

* labelled DAG
* Nodes: literals in partial assignment
* Node label T: indicates assignment and decision level
* Edges: from l1...lk to l labelled with C Assignments l1...lk caused assignment to l during BCP
* Special node C: conflict node

#### MiniSAT
##### watch points
* obvious/naive implementation
    * mark all satisfied clauses
    * maintain a count of non-false literals (detect literal clauses) in every clause
    * for each literal, maintain list of obvious if appears in
* every clause two watch points
    * `(p v q v r v s)`
* each variable has two lists
    1. clauses with a watch pointer to its true literal
* invariant: if a clause in unsatisfied watch points point to two distinct literals, neither of which have set to false
    * when new variable assigned only clauses one ahe of its two lists we searched
    * e.g., if variable assigned T -> clauses with negative watch will be violated

Dynamic largest individuals sum (DLIS): choose literal satisfying largest number of clauses

Variable state independent decaying sum (VSIDS): favor literals occurring in a lot of conflicts: +1 (pick the highest scrore to choose)

#### MaxSAT
given formula F (CNF): maximize the number of satisfied clauses

#### Partial MaxSAT
`F ^ Q; m |= F, m |= Q*`

### First-order logic
    C object constants
    F function constatns -> function f(a) -> b
    R relation constants -> P(a) > (0, -1)
    C, F, R -> signature of the logic
    L(C, F, R) = first-order language

    Basic terms: a constant c belongs to C or a variable (x, y, z ...)
    Composite terms: f(t1, t2 ... tk), arity of f is k, e.g., age(mother(mvc))

    Formular: atomic predicator: p(t1, t2 ... tk)
    F1, F2 -> F1 ^ F2, F1 v F2
    F -> !F

    given F:
        Vx. F
        Ex. F # F is scope of quantifier
        we can nest predicates and functions

    Vy.((Vx.p(x)) -> q(x, y))
    closed formula: (no free variables)
    ground formula: (no variables) > (0, -1)

#### Semantics
* U, universe of discourse: a non-empty set of objects
* An interpretation I is a mapping from C, F, R to objects from U

        I maps every c belongs to C to I(c) belongs to U
        I maps every n-arg f belongs to F to fi: Un -> U
        I maps every n-arg P belongs to R to pi belongs to U


    objects {a, b, c}
    function f (binary)
    relation r (3-arg)
    universe = {1, 2, 3}
    I(a) = 1, I(b) = 2, I(c) = 3
    I(f) = {1->2, 2->3, 3->1}
    I(r) = {(1,2,3), (3,2,1)}



Structure(algebra): S = (U, I) for a first-order laguary L(L, F, R)

A variable assignment: 6
    mapping from variables to U
    U = {#, O}, 6(x) = #

#### Evaluation
    terms: <I, G>(t)
        Object<I, 6>(a) = I(a)
        Variable<I, 6>(v) = 6(v)
        Function<I, G>(f(t1, t2 ... tk)) = I(f)(<I,6>(t1) ... tk)

    evaluation: (satisfy)
        U, I, 6 |= F
        U, I, 6 |!= F
        U, I, 6 |= Va.F iff for all o belongs to U, U,I,6[x->o] |= F
        U, I, 6 |= Ex.F iff for some o belongs to U, U,I,6[x->o] |= F

    F is SAT iff there exists some S and 6
        s.t. S,6 |= F
    A structure S is a model of F if
        for all 6 we have S,6 |= F

    F is VALID iff !F is UNSAT
    semantics argument method
    S,6 |= !F   S,6 |!= !F
    S,6 |!= F   S,6 |= F

    Universal
        U,I,6 |= Vx.F
    --------------------- (for any o belongs to U)
        U,I,6[x->o] |= F

        U,I,6 |!= Vx.F
    --------------------- (a fresh o belongs to U) (fresh: new, have not used before)
        U,I,6[x->o] |!= F

        U,I,6 |= Ex.F
    --------------------- (a fresh o belongs to U)
        U,I,6[x->o] |= F

        U,I,6 |!= Ex.F
    --------------------- (for any o belongs to U)
        U,I,6[x->o] |!= F

    U,I,6 |= P(s1, s2 ... sn)
    U,I,6 |!= P(t1, t2 ... tn)
    <I,6>(si)=<I,6>(ti) Vi belongs [1, n]
    -------------------------------------
        U,6,I |= 止

##### Soundness
if every branch delivers a contradiction then F is valid

##### Complete
if F is valid then there is a finite length proof of 止


##### FOL validity is undecidable
FOL is semidecidable: if a formula is VALID then we can prove that, otherwise we will go on forever

#### Decidable fragments
1. quantifier-free FOL -> SAT (NP-complete)
- monadic FOL
    * pure: all predicates take 1 argument has functions and letters
    * impure: allow monadic functions
- [Bernays–Schönfinkel class] (http://en.wikipedia.org/wiki/Bernays%E2%80%93Sch%C3%B6nfinkel_class)
    1. no function constants
    - only formulates of the farm

#### [Datalog] (http://en.wikipedia.org/wiki/Datalog)
all clauses are [Horn clauses] (http://en.wikipedia.org/wiki/Horn_clause)
