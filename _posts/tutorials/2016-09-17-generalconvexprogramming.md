---
title: "General convex programming"
category: tutorial
tags:
level: 3.2
excerpt: "YALMIP does not care, but for your own good, think about convexity and structure also in general nonlinear programs."
sidebar:
  nav: "tutorials"
---

YALMIP really does not care if your general nonlinear program is convex or not. In the general case it will just be a nonlinear model and any nonlinear solver is used. If it happens to be convex, the solver might perform better, but that is not something YALMIP can influence.

However, all nonconvex models are not created equal. There are nice convex models, and nasty nonconvex models, from YALMIPs perspective.

## Analytic center of polytope - bad vs good forms

As an example, we will find the analytic center of a polytope \\(Ax \leq b\\). The analytic center is defined as the point which maximizes the expression \\( \sum \log(b-Ax)\\)

Define a polytope and the decision variable.

````matlab
n = 5;m = 20;
A = randn(m,n);
b = rand(m,1)*m;
x = sdpvar(n,1);
````

Solve the problem using the overloaded concave [log](/command/log).

````matlab
optimize(A*x <= b,-sum(log(b-A*x)))
````

If you have an [exponential cone programming solver](/tags/#exponential-cone-programming-solver) installed, this will solve nicely. However, if YALMIP has to revert to a [standard nonlinear solver](/tags/#nonlinear-programming-solver), it can easily fail. The reason is that these solvers often have problems with models where the objective function is undefined for infeasible points, or more generally have singularities.

If you have problems related to this on your model, we can use the exponential operator instead and solve an inverse formulation of the same problem.

````matlab
y = sdpvar(m,1);
optimize(exp(y) <= b-A*x,-sum(y));
````

A completely equivalent problem with much better properties in a general nonlinear solver. Note that also this model will be solved with an [exponential cone programming solver](/tags/#exponential-cone-programming-solver) if available.


By default, YALMIP allows nonconvex problems to be formulated. However, if you want to make sure that no nonconvex problems slips by, you can specify this. If we switch the sign on the objective a non-convex model is obtained, and YALMIP detects this.

````matlab
optimize(A*x <= b,sum(log(b-A*x)),sdpsettings('allownonconvex',0))
````
