---
category: faq
title: "Duals from second-order cone or quadratic model"
tags: [Duality, Dual]
date: '2021-04-07'
sidebar:
  nav:
---

When it comes to extracting duals for second-order cone programs or quadratically constrained programs, it depends on both solver and the way you modelled the constraint.

## CPLEX

The interface YALMIP uses to communicate with [CPLEX](/solver/cplex) (*cplexqcp.m*) does not support extraction of SOCP duals.

## Gurobi

To extract duals from second-order cone constraints using [GUROBI](/solver/gurobi), the functionality must be activated in the solver first. The solver supports both extraction of the conic vector dual on [CONE](/command/cone) constraints, and extraction of a dual to a scalarized quadratic constraint.

````matlab
x = sdpvar(2,1);
Model = [x'*x <= 1];
options = sdpsettings('solver','gurobi','gurobi.qcpdual',1);
optimize(Model,sum(x),options);
dual(Model(1))

ans =

    0.7071

Model = [cone([1;x])];
options = sdpsettings('solver','gurobi','gurobi.qcpdual',1);
optimize(Model,sum(x),options);
dual(Model(1))

ans =

    1.4142
    1.0000
    1.0000

````

## Mosek

With [MOSEK](/solver/mosek), only socp duals on [CONE](/command/cone) constraints are possible.

````matlab
x = sdpvar(2,1);
Model = [cone([1;x])];
options = sdpsettings('solver','mosek');
optimize(Model,sum(x),options);
dual(Model(1))

ans =

    1.4142
    1.0000
    1.0000

````
