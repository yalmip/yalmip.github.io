---
layout: single
permalink: /interp1qp
excerpt: "Convenient generation of approximations"
title: "Nonconvex QP via piecewise affine models"
tags: [Nonconvex quadratic programming, sos2, interp1]
comments: true
date: '2017-09-16'
---

To showcase the generality and convenince of the [interp1](/command/interp1), let us answer a common question which addresses the problem of solving integer quadratic programs using linear solvers, and to make matters worse, we study indefinite quadratic objectives.

The simple idea we will use is to approximate the quadratic function as a piecewise affine function. Of course, this is not necessarily a good way to solve indefinite quadratic programs, but it is a common strategy ([see this post](/example/nonconvexquadraticprogramming) for some alternatives). Let us assume we want to minimize the indefinite objective \\(x'^TQx - y^TRy\\) over the unit-box intersected with \\(\sum x + \sum y = 1\\).

````matlab
n = 10;
x = sdpvar(n,1);
y = sdpvar(n,1);
Q = randn(n);Q = Q*Q';
R = randn(n);R = R*R';
Model = [-1 <= [x y] <= 1, sum(x) + sum(y) == 1];
````

To begin with, a problem here is that the model is multivariate, but the [interp1](/command/interp1) only acts on univariate data. To handle that, we factorize the quadratic functions so we can write them as sums of univariate functions \\(\sum e_i^2 - \sum f_i^2\\)
````matlab
S = chol(Q);
T = chol(R);
e = sdpvar(n,1);
f = sdpvar(n,1);
Model = [-1 <= [x y] <= 1, sum(x) + sum(y) == 1, e == S*x, f == T*y];
````

Our next step is to introduce a piecewise affine approximation of every quadratic term \\(e_i\\) and \\(f_i\\) using [interp1](command/interp1). To do this, we have to define the domain over which the functions are approximated, i.e., find lower and upper bounds on the elements in \\(e\\) and \\(f\\). We can conviently do that using [boundingbox](/command/boundingbox)

````matlab
[~,Le,Ue] = boundingbox(Model,[],e);
[~,Lf,Uf] = boundingbox(Model,[],f);
````

Now, generate a grid over the bounding box and define the piecewise affine approximators

````matlab
N = 100;
E = repmat(Le,1,N) + repmat(linspace(0,1,N),n,1).*repmat(Ue-Le,1,N);
F = repmat(Lf,1,N) + repmat(linspace(0,1,N),n,1).*repmat(Uf-Lf,1,N);

f1 = interp1(E,E.^2,e,'lp');
f2 = interp1(F,F.^2,f,'lp');
````

With the flag **'lp'**, the way the interpolation is implemented depends on data and convexity propagation. An efficient linear programming based graph representation will be used if possible, while a mixed-integer [sos2](/command/sos2) approach is used otherwise. In our case, the first term is convex and will thus be implemented efficiently, while the second term requires  [sos2](/commandsos2)

````matlab
optimize(Model,sum(f1)-sum(f2))
````

If we have a convex mixed-integer quadratic programming solver, there is no need to approximate the first convex part of the objective, so we can use a partially quadratic model instead
````matlab
Model = [-1 <= [x y] <= 1, f == T*y];
optimize(Model,x'*Q*x-sum(f2))
````
