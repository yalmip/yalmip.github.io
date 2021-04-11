---
layout: single
category: faq
author_profile: false
excerpt: 
title: "Warning about lousy big-M models"
tags: [Big-M, Implies]
comments: true
date: '2021-04-11'
sidebar:
  nav:
---

You are setting up a model which YALMIP will model using a [big-M model](/tutorial/bigmandconvexhulls) (logical models involving implications, or nonconvex use of MILP-representable operators etc), and you see warnings about unbounded variables leading to lousy big-M models. in your mind, everything is bounded though, so what's up?

## Explicit variable bounds vs implicit variable bounds

As an example, consider the following trivial model

````matlab
u = sdpvar(1);
x = sdpvar(1);
d = binvar(1);
Model = [implies(d, u-x == 0),         
         1 <= u <= 2, 
         y == 0,
         x == y
         d == 0]
optimize(Model)
````

When we look at the model, we immediately derive bounds on all variables. The variables **u**, **d** and **y** are **explicitly** bounded in the model through variable bounds or variable equalities. The variable **x** is not explicitly bounded, but **implicitly** we trivially see its bound as it is equal to the bounded variable **y**. 

The problem though is that YALMIP does not perform any bound propagation (i.e. deriving implicit bounds) when extracting the bounds in the model which are used to create the big-M model of the implication. This means the variable **x** has no detected bounds. The big-M model of the implication is \\( -M(1-d) \leq u-x \leq M(1-d) \\) where \\(M\\) is an upper bound on \\(u-x\\). Since YALMIP has no bound available on **x**, it cannot derive a bound on \\(u-x\\), and will simply use the default value \\(M=10^4\\). When the reformulation engine sees this value it will warn since it indicates lack of (explicit) bounds and large big-M constants like this easily leads to poor models.

Add a bound and everything works

````matlab
optimize([Model, -10 <= x <= 10])
````

## Poor explicit bounds
If you add a huge bound though, nothing improves and you will still see the warning since your model once again will have huge bounds on terms in big-M representations (remember:big-M should really be called *as small as possible but sufficiently large-M*)

````matlab
optimize([Model, -10^5 <= x <= 10^5])
````

## Bad scaling of model leading to infeasibility due to bad defaults

Things can get even worse. Had you started with the following model you would also have warnings. However, now the derived model turns out to be infeasible (although we manually can see that the problem is feasible)

````matlab
u = sdpvar(1);
x = sdpvar(1);
d = binvar(1);
Model = [implies(d, u-x == 0),         
         1e5 <= u <= 2e5, 
         y == 0,
         x == y
         d == 0]
optimize(Model)
````

Since **d** is zero, the implication should be inactive. The problem is once again that no bound is available on **x** so the big-M model for the implication will be \\(  -10^4(1-d) \leq u-x \leq 10^4 (1-d)\\). With the feasible values on **x** and **d** this means essentially says \\( -10^4 \leq u \leq 10^4\\) which is inconsistent with \\(10^5 \leq u \leq 2\cdot 10^5\\). The default value is simply not sufficiently large on this badly scaled model. 

Hence, YALMIP assumes that your model is nicely scaled, and that you have explicit bounds on all variables encountered in big-M represented expressions.

## Bounds hidden in the modelled logics

A final example is the case when explicit bounds are in the model, but they are hidden inside logics.

````matlab
x = sdpvar(1);
Model = [implies(d,   -1 <= x <= 1),         
         implies(1-d, -2 <= x <= 2)];
optimize(Model)
````

Obviously **x** is bounded in any feasible solution, and we see what appears to be explicit bounds on **x**. However this is only because we manually presolve the model when we look at it. We immediately solve the MILP and see that there are two cases and that the union of these two possibilities gives us trivial bounds. This is not possible when YALMIP searches for the bounds in the model as the bounds effectively are hidden inside a MILP representation, and that MILP representation cannot be constructed until bounds are available. A chicken-and-egg problem. To be able to detect these bounds a full-fledged high-level MILP preolve routine would have to be engaged.

