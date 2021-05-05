---
category: inside
permalink: multiplesolutions
excerpt: "Avoid that for-loop by using vector objectives"
title: Computing multiple solutions in one shot
date: '2009-08-29'
---

The overhead from YALMIPs symbolic manipulations and generation of numerical data for a solver can in some cases be severe. A typical case is when we want to solve an optimization problem for a series of objective functions. In some cases, this can be reduced significantly by using YALMIP support for multiple objectives and associated multiple solutions. Note that this functionality currently only works for linear objectives.

Consider a manual implementation of the [boundingbox](/command/boundingbox) command

````matlab
x = sdpvar(3,1);
Ball = x'*x <= 1;

ops = sdpsettings('verbose',0);
for i = 1:3
  optimize(Ball,x(i),ops);
  L(i) = value(x(i));
  optimize(Ball,-x(i),ops);
  U(i) = value(x(i));
end
````

An alternative approach is to give several objective functions at once to YALMIP, and thus reduce the overhead from analyzing the problem and looking for suitable solvers etc. After solving the problem with 6 different objective functions, YALMIP will have 6 solutions available, and a particular solution is extracted by using a second argument in the [value](/command/value) operator.

````
optimize(Ball,[x;-x],ops);
for i = 1:3
  L(i) = value(x(i),i);
  U(i) = value(x(i),i+3);
end
````

Alternatively, a specific solution can be made active globally.

````matlab
optimize(Ball,[x;-x],ops);
for i = 1:3
  selectsolution(i);
  L(i) = value(x(i));
  selectsolution(i+3);
  U(i) = value(x(i));
end
````

An alternative approach to compute many solutions with small differences in the setup is to use an [optimizer approach](/command/optimizer)
