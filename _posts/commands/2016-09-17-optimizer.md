---
layout: single
category: command
author_profile: false
excerpt: "Create precompiled optimization model object"
title: optimizer
tags:
comments: true
date: '2016-09-17'
sidebar:
  nav: "commands"
---

[optimizer] creates an object with an exported optimization model which can be used to efficiently solve a series of similar problems.

### Syntax

````matlab
P = optimizer(Con,Obj,Options,Parameters,WantedVariables)
````

### Examples

[optimizer] is used to simplify and speed up code where the almost the same model is setup and solved repeatedly. 

As a start, we create a trivial linear programming model where a scalar decision variable is bounded from below by some value \\(a+1\\). We create an optimizer object where the bound \\(a\\) is considered a parameter, and we are interested in the minimal value of \\(x^2\\) as the parameter \\(a\\) varies.


````matlab
sdpvar a x
Constraints = [a+1 <= x];
Objective = x^2;
P = optimizer(Constraints,Objective,[],a,x)
Optimizer object with 1 inputs and 1 outputs. Solver: MOSEK-LP/QP
````

Solve the problem for the case when \\(a=1\\).


````matlab
P{1}
ans =

    2.0000
````

Solve the problem for a range of parameters

````matlab
z = (-5:0.1:5);
plot(z,P{z})
````

Effectively, when the model is affinely parameterized in a parameter, a precompiled numerical model is created, and when a solution for a particular parameter value \\(a^{\star}\\) is requested, the precompiled model is appended with the constraint \\(a = a^{\star}\\) and sent to the solver.

See more examples inn the [Examples.StandardMPC MPC example] and  [Examples.UnitCommitment unit commitment example].

### Comments

Note that assigned values of [sdpvar] objects are not updated after the optimization problem is solved.

[Tutorials.SumOfSquares Sum-of-squares] problems can be handled through optimizer also. Note though that parameters in the sum-of-squares problem cannot be explicitly defined in [Commands.Optimizer optimizer], but YALMIP has to deduce them from non-sos constraints, the objective, input parameters and output parameters. If you first manually apply [Commands.Compilesos compilesos] to extract an SDP representation, you must set the option '''sos.model''' to '''2''' (image representation, dual parameterization) to be able to return output values.

Variables involved in defining the geometry of an uncertainty set when using the [Tutorials.RobustOptimization robust optimization framework] cannot be a parameter (during compilation, YALMIP treats all parameters as  decision variables, and this effectively means that there is no description of the uncertainty set (the uncertainty set is defined as the constraints only involving uncertain variables). Hence, the following scaled box will not work

````matlab
sdpvar t U w
P = optimizer([uncertain(w), -U <= w <= U , b*w <= t],t,[],U,t)
````
Many times, this can be fixed by introducing normalized uncertainty sets and scaled uncertainties instead.

````matlab
sdpvar t U wnorm
w = wnorm*U:
P = optimizer([uncertain(wnorm), -1 <= wnorm <= 1 , b*w <= t],t,[],U,t)
````

### See also
[optimize], [export], [solvemp]
