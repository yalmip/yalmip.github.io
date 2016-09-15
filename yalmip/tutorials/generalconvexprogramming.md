---
title: "General convex programming"
layout: single
sidebar:
  nav: "tutorials"
---

YALMIP is foremost meant to be used as a modelling tool for conic optimization problems, or problems that can be converted to conic optimization problems. However, by using the [[Tutorials.NonlinearOperators#evaluationbased | evaluation based nonlinear operator]] framework, it is possible to solve also general convex optimization problems (and nonconvex). At the moment, this feature adds support for most operators available in MATLAB, such as logarithms and exponentials, and can easily be extended, as explained in the [[Tutorials.NonlinearOperators#addevaluationbased | nonlinear operator tutorial]].

You are advised to read the [[Tutorials.NonlinearOperators | nonlinear operator tutorial]] before reading this tutorial.

As an example, we will find the analytic center of a polytope \\(Ax\leq b\\). The analytic center is defined as the point which maximizes the expression \\( \sum \log(b-Ax)\\)

Define a polytope and the decision variable.
````matlab
n = 5;m = 20;
A = randn(m,n);
b = rand(m,1)*m;
x = sdpvar(n,1);
````

Solve the problem using the overloaded concave [[Commands.Log | log operator]] (for this to work, you need to have a general purpose nonlinear solver installed)
````matlab
optimize(A*x <= b,-sum(log(b-A*x)))
````

Most likely, this will fail if you try to run it. The reason is that nonlinear solvers often have problems with models where the objective function is undefined for infeasible points.

To avoid this, we can use the exponential operator instead and solve an inverse formulation of the same problem.
````matlab
y = sdpvar(m,1);
optimize(exp(y) <= b-A*x,-sum(y));
````

Finally, note that we can solve the problem using the overloaded [[Tutorials.NonlinearOperators | geomean]] operator. This will however lead to a second order cone problem.
````matlab
optimize([],-geomean(b-A*x));
````

By default, YALMIP allows nonconvex problems to be formulated. However, if you want to make sure that no nonconvex problems are set up by YALMIP, you can specify this
````matlab
optimize(A*x <= b,sum(log(b-A*x)),sdpsettings('allownonconvex',0))
````
