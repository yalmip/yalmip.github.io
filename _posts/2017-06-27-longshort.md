---
layout: single
excerpt: "There is more than one way to skin a cat"
title: Nonconvex long-short constraints
tags: [Integer programming, Logic programming, Cardinality]
comments: true
date: 2017-06-27
---

A question on the forum today asked how one can constrain a solution to ensure a certain percentage of a vector to have a particular sign. This could for instance be of relevance in a [portfolio allocation](/example/portfolio) problem where we have constraints on the ratio of long and short positions.

There are many natural ways to write this in MATLAB, but to include it in YALMIP code, it has to be based on operators which are supported by YALMIP. Let's look at some alternatives.

Let us assume we have a vector \\(x\\) of length \\(n\\) and we want at least half of the elements to be non-negative. To have a problem to work with, we will minimize the distance to a vector \\(y\\).

````matlab
n = 20;
x = sdpvar(n,1);
y = randn(n,1);
Objective = (x-y)'*(x-y);
````

To begin with, one has to understand that the constraint is nonconvex (it is a combinatorial constraint), and the end result in YALMIP will be a [mixed-integer representation](/tutorial/nonlinearoperatorsmixedinteger) of the constraint, more precisely a [big-M representation](/tutorial/bigmandconvexhulls/). 

For the experienced user, a manual big-M model is obvious. Define \\(n\\) binary variables \\(d\\), where \\(d_i = 1\\) indicates that \\x_i\\) is non-negative, and the model is straightforward. For future reference, we introduce a global bound on \\(x\\) as it will be required later when YALMIP is supposed to derive a sufficient big-M constant.

````matlab
d = binvar(n,1)
M = 1;
Model = [x >= -(1-d)*M, sum(d) >= 0.5*n, -1 <= x <= 1];
optimize(Model,Objective)
````

What we have implemented here is essentially an implication between the binary vector **d** and **x**. This can more conveniently be written using [implies](/command/implies)

````matlab
d = binvar(n,1)
Model = [implies(d,x>=0), sum(d) >= 0.5*n,  -1 <= x <= 1];
optimize(Model,Objective)
````

Going beyond these two models is never adviced, as it only complicates matters. However, let's see if we can come up with other models just for fun.

The operator [sort](/command/sort) is overloaded, and our constraint can be states as the last \\(n/s\\) elements of the sorted version of \\(x\\) should be non-negative. This will lead to a much worse integer model so never ever use the sort operator unless you have to

````matlab
sorted = sort(x);
Model = [sorted(1:n/2) >= 0,  -1 <= x <= 1];
optimize(Model,Objective)
````

Another approach is to use cardinality and the [nnz](/command/nnz) applied to constraints.

````matlab
Model = [nnz(x >= 0) >= n/2,  -1 <= x <= 1];
optimize(Model,Objective)
````



