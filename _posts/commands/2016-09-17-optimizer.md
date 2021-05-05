---
category: command
excerpt: "Create precompiled optimization model object"
title: optimizer
tags:
date: '2016-09-17'
sidebar:
  nav: "commands"
---

[optimizer](/command/optimizer) creates an object with a pre-compiled low-level numerical format which can be used to efficiently solve a series of similar problems (reduces YALMP analysis and compilation overhead)

See the post [updated optimizer](/optimizerupdates) for recent extensions and improvements.

### Syntax

````matlab
P = optimizer(Con,Obj,Options,Parameters,WantedVariables)
````

### Examples

First a word of warning: Never start your model development using optimizer constructs. Always start with a standard model based on calls to [optimize](/command/optimize), and then when everything works as expected, adjust you model to use [optimizer](/command/optimizer) instead, if you think this will improve simulation performance. Debugging models in the [optimizer](/command/optimizer) world is much harder and not recommended.

[optimizer](/command/optimizer) is used to simplify and speed up code where  almost the same model is setup and solved repeatedly.

As a start, we create a trivial linear programming model where a scalar decision variable \\(x\\) is bounded from below by some value \\(a+1\\). We create an optimizer object where the bound \\(a\\) is considered a parameter, and we are interested in the minimizing argument of \\(x^2\\) as the parameter \\(a\\) varies.


````matlab
sdpvar a x
Constraints = [a+1 <= x];
Objective = x^2;
P = optimizer(Constraints,Objective,[],a,x)
Optimizer object with 1 inputs and 1 outputs. Solver: MOSEK-LP/QP
````

Solve the problem for the case when \\(a=1\\).


````matlab
P(1)
ans =

    2.0000
````

Solve the problem for a range of parameters

````matlab
z = (-5:0.1:5);
plot(z,P(z))
````

Effectively, when the model is affinely parameterized in a parameter, a precompiled numerical model is created, and when a solution for a particular parameter value \\(a^{\star}\\) is requested, the precompiled model is appended with the constraint \\(a = a^{\star}\\) and sent to the solver.

The case when the model is nonlinearly parameterized is handled in a different way. Since the precompiled model is nonlinear in the parameter, simply adding an equality constraint will not work, as the model then would be nonlinear. Instead, YALMIP performs a variable elimination on the precompiled model, and the reduced model is sent to the solver.

Consider the following nonlinearly parameterized quadratic program

````matlab
sdpvar a x
Constraints = [-1 <= x <= -a^2/25];
Objective = a*x^2 + 2*x;
P = optimizer(Constraints,Objective,[],a,x)
Optimizer object with 1 inputs and 1 outputs. Solver: FMINCON-STANDARD
````

Note that YALMIP has picked a nonlinear solver. The reason is that YALMIP does not exploit the knowledge that \\(a\\) is a parameter, when the model is compiled. Hence, it thinks the objective is cubic and the constraints are quadratic, and picks a general nonlinear solver.

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
plot(P(0:.1:5))
````

Note that the solver selected here is a convex QP solver. Hence, it is not applicable when \\(a \le 0 \\). The behaviour for this case is undefined, and it is up to you to select a suitable solver for the parameter values you will see.

Error flags from the solutions can be extracted using a second argument

````matlab
[xvalue, errorcode] = P(pi)
````

By default, the solver is run in silent model, and to turn on display we have to increase verbosity level to 2.

````matlab
options = sdpsettings('solver','mosek','verbose',2);
P = optimizer(Constraints,Objective,options,a,x)
[xvalue, errorcode] = P(pi)
````

If you have many parameters and outputs, you can of course vectorize the 4th and 5th arguments suitably, but a more convenient way is to use a cell format

````matlab
sdpvar a b x y
Constraints = [-1 <= x <= -a^2/25 + b*y];
Objective = a*x^2 + 2*x+y^2;
options = sdpsettings('solver','mosek');
P = optimizer(Constraints,Objective,options,{a,b},{x,Objective})
[sol, errorcode] = P({pi,1});
[sol{1} sol{2}]
% Alternative
[sol, errorcode] = P(pi,1);
[sol{1} sol{2}]
````


See more examples in the [MPC example](/example/standardmpc) and  [unit commitment example](/example/unitcommitment).

### Comments

Note that assigned values of [sdpvar](/command/sdpvar) objects are not updated after the optimization problem is solved.

 Sum-of-squares problems can be handled through optimizer also. Note though that parameters in the sum-of-squares problem cannot be explicitly defined in [optimizer](/command/optimizer), but YALMIP has to deduce them from non-sos constraints, the objective, input parameters and output parameters.
 
 Consider the problem of finding a lower bound on a polynomial in a variable \\(x\\) using [sum-of-squares](/command/solvesos). Here, \\(t\\) is automatically detected as a parameter as it is part of the objective
 
 ````matlab
sdpvar x t
p = 1 + x + x^2 + x^3 + x^4
solvesos(sos(p-t),-t)
````

Now consider the case where we want to investigate a lower bound for a set of polynomials with undeceided variables. Effectively, we want to find a parameter \\(b\\) such that the lower bound is maximized, and test this for a sequence of polynomials depending on a parameter \\(a\\) (which thus is fixed for every sum-of-squares problem solved). 

 ````matlab
sdpvar x t a b
p = 1 + b*x + b*x^2 + a*x^3 + x^4
````

To ensure YALMIP understands that the sum-of-squares decomposition is performed over \\(x\\) only, we must add \\(b\\) to the parametric part of the sum-of-squares model, or add it to the list of outputs from optimizer. YALMIP automatically understands that \\(t\\) is a parameter as it is part of the objective (and declared as an input parameter to optimizer)

 ````matlab
P = optimizer([sos(p-t), -1000 <= b <= 1000],-t,sdpsettings('solver','mosek'),a,t);
plot(P(-10:1:10))
% Alternatively
P = optimizer(sos(p-t),-t,[],a,[t;b]);
sol = P(-10:1:10);
plot(sol(1,:))
````

Variables involved in defining the geometry of an uncertainty set when using the robust optimization framework cannot be parameters (during compilation, YALMIP treats all parameters asdecision variables, and this effectively means that there is no description of the uncertainty set (the uncertainty set is defined as the constraints only involving uncertain variables)). Hence, the following scaled uncertainty box will not work

````matlab
sdpvar t U w
P = optimizer([uncertain(w), -U <= w <= U , b*w <= t],t,[],U,t)
````

Many times, this can be fixed by introducing normalized uncertainty sets and scaled uncertainties instead.

````matlab
sdpvar t U wnormalized
w = wnorm*U:
P = optimizer([uncertain(wnormalized), -1 <= wnormalized <= 1 , b*w <= t],t,[],U,t)
````
