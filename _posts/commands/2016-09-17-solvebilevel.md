---
layout: single
category: command
author_profile: false
excerpt: "Built-in bilevel optimization problem solver"
title: solvebilevel
tags: [Bilevel programming]
comments: true
date: '2016-09-17'
sidebar:
  nav: "commands"
---

[solvebilevel](/command/solvebilevel) is a built-in bilevel solver for problems with inner convex quadratic programs.

### Syntax

````matlab
[sol,info] = solvebilevel(OConstr,OObj,IConstr,IObj,IVar,options)
````

### Examples

By default, the solver is based on a simple branching strategy, where branching is performed on complementarity of duals and slacks in the KKT conditions. Hence, the solver is only applicable to small academic examples. It has however been shown in experiments, that the solver performs fairly well in some cases compared to a [big-M reformulation](example/bilevelprogrammingalternatives) followed by a solution using a mixed-integer solver such as [CPLEX](/solver/cplex) or [GUROBI](/solver/gurobi). Since no [big-M](/tutorial/bigmandconvexhulls/) numbers are used, it is much more numerically robust in some cases where no reasonable bounds can be derived on dual variables.


````matlab
sdpvar x1 x2 y1 y2 y3

OO = -8*x1-4*x2+4*y1-40*y2-4*y3;
CO = [x1>=0, x2>=0];

OI = x1+2*x2+y1+y2+2*y3;
CI = [[y1 y2 y3] >= 0,
       -y1+y2+y3 <= 10,
      2*x1-y1+2*y2-0.5*y3 <= 10,
      2*x2+2*y1-y2-0.5*y3 <= 9.7];

solvebilevel(CO,OO,CI,OI,[y1 y2 y3])

>> value([y1 y2 y3])

ans =

         0    6.0000    4.0000

>> value([x1 x2])

ans =

         0    8.8500
````

As an alternative, we can tell the solver to derive the [KKT conditions](/command/kkt), and model the complementary slackness conditions using a big-M approach, and solve the problem using a standard mixed-integer solver.

````matlab
ops = sdpsettings('bilvel.algorithm','external');
solvebilevel(CO,OO,CI,OI,[y1 y2 y3])
````
If you would like to use this approach, you are however recommended to derive the problem by calling the [KKT](/command/kkt) operator and setup the problem manually, in order to have full control of the way you work with bounds on the dual variables.
