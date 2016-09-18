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

[optimizer] creates an object with an low-level numerical format which can be used to efficiently solve a series of similar problems (reduces YALMP analysis and compilation overhead)

### Syntax

````matlab
P = optimizer(Con,Obj,Options,Parameters,WantedVariables)
````

### Examples

[optimizer] is used to simplify and speed up code where the almost the same model is setup and solved repeatedly. 

As a start, we create a trivial linear programming model where a scalar decision variable \\(x\\) is bounded from below by some value \\(a+1\\). We create an optimizer object where the bound \\(a\\) is considered a parameter, and we are interested in the minimal value of \\(x^2\\) as the parameter \\(a\\) varies.


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

The case when the model is nonlinearly parameterized is handled in a different way. Since the precompiled model is nonlinear in the parameter, simply adding an equality constraint will not work, as the model then would be nonlinear. Instead, YALMIP perform a variable elimination on the precompiled model, and the reduced model is sent to the solver.

Consider the following nonlinearly parameterized quadratic program
````matlab
sdpvar a x
Constraints = [-1 <= x <= -a^2/25];
Objective = a*x^2 + 2*x;
P = optimizer(Constraints,Objective,[],a,x)
Optimizer object with 1 inputs and 1 outputs. Solver: FMINCON-STANDARD
````

Note that YALMIP has picked a nonlinear solver. The reason is that YALMIP does not exploit the knowledge that a is a parameter, when the model is compiled. Hence, it thinks the objective is cubic and picks a general nonlinear solver.

To circumvent this, we have to tell YALMIP which solver we want to use, and thus promise YALMIP that this solver actually is capable of solving the problem once the parameters have been fixed.

````matlab
sdpvar a x
Constraints = [-1 <= x <= -a^2/25];
Objective = a*x^2 + 2*x;
options = sdpsettings('solver','mosek');
P = optimizer(Constraints,Objective,options,a,x)
````

We can now use the optimizer object as usual

````matlab
plot(P{[0:.1:5]})
````

Note that the solver selected here is a convex QP solver. Hence, it is not applicable when \\(a \le 0 \\). The behaviour for this case is undefined, and it is up to you to select a suitable solver for the parameter values you will see.

Error flags from the solutions can be extracted using a second argument

````matlab
[xvalue, errorcode] = P{[pi]})
````


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
