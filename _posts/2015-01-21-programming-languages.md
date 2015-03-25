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

Implication graph: lead to a conflict, subgraph characterizes the conflicts

* labelled DAG
* Nodes: literals in partial assignment
* Node label T: indicates assignment and decision level
* Edges: from l1...lk to l labelled with C Assignments l1...lk caused assignment to l during BCP
* Special node C: conflict node
