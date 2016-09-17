---
title: "General convex programming"
type: tutorial
author_profile: false
tags:
excerpt: "YALMIP does not care, but for your own good, think about convexity also in general nonlinear programs."
layout: single
sidebar:
  nav: "tutorials"
---

YALMIP was initially developed as a modelling tool for conic optimization problems, or problems that can be converted to conic optimization problems. However, today it is a rather general modelling tool with support for most functions and operators.

You are advised to read the [nonlinear operators tutorial](/yalmip/tutorials/nonlinearoperators) before reading this tutorial.

As an example, we will find the analytic center of a polytope \\(Ax \leq b\\). The analytic center is defined as the point which maximizes the expression \\( \sum \log(b-Ax)\\)

Define a polytope and the decision variable.

````matlab
n = 5;m = 20;
A = randn(m,n);
b = rand(m,1)*m;
x = sdpvar(n,1);
````

Solve the problem using the overloaded concave [log] (for this to work, you need to have a general purpose nonlinear solver installed)

````matlab
optimize(A*x <= b,-sum(log(b-A*x)))
````

Most likely, this will fail if you try to run it. The reason is that nonlinear solvers often have problems with models where the objective function is undefined for infeasible points.

To avoid this, we can use the exponential operator instead and solve an inverse formulation of the same problem.

````matlab
y = sdpvar(m,1);
optimize(exp(y) <= b-A*x,-sum(y));
````

Finally, note that we can solve the problem using the overloaded [geomean] operator. This will however lead to a second order cone problem.

````matlab
optimize([],-geomean(b-A*x));
````

By default, YALMIP allows nonconvex problems to be formulated. However, if you want to make sure that no nonconvex problems are set up by YALMIP, you can specify this

````matlab
optimize(A*x <= b,sum(log(b-A*x)),sdpsettings('allownonconvex',0))
````
