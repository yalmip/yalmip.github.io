---
layout: single
permalink: /Initials
excerpt: "Give your solver a hint"
title: "Supplying initial guesses to solvers"
tags: [Warm-start, Initial]
comments: true
date: '2017-08-25'
---

In some cases, the solver might benefit from an initial guess on where to start the search, so called warm-starting.

## Linear, quadratic and conic programming

Before we start supplying inititial guesses to solvers, it should be mentioned that when you solve simple linear, convex quadratic, second-order cone or linear semidefinite programs, there is typically no need to supply initial guesses. 

These solvers tyically work with your model in a primal-dual space, often with a reformulated model compared to your high-level model, and you will have no idea about a relevant initial primal-dual pair. In fact, many of these solvers have no way to supply an initial guess, and if they had, the solution-time would most likely not change much. Warm-starting these solvers is a field of research, and to a large degree, not general efficient methods are available.

## Nonlinear optimization

The typical situation where you would want to supply an anitial guess is when you use a general purpose nonlinear solver, such as [fmincon](/solver/fmincon), [knitro](/solver/knitro) or [ipopt](/solver/ipopt). 

Supplying initial gueeses are typically done for two reasons. The first and most common reason is that we have a vague idea about where the solution to a non-convex problem is, and want to start the search in that region. The second reason is that the problem involves singularities and generally tricky regions, and we want to ensure the solver does not start there.

As a simple example, consider the following trivial convex program

````matlab
sdpvar x
Constraints = [x <= 1];
Objective = 1/x
````

If we try to solve this problem using [fmincon](/solver/fmincon) it will crash. To see the crash clearly, we turn on debug mode

````matlab
optimize(Constraints,Objective,sdpsettings('debug',1))
Error using sfminbx (line 28)
Objective function is undefined at initial point. fmincon cannot continue.

Error in fmincon (line 889)
      [X,FVAL,LAMBDA,EXITFLAG,OUTPUT,GRAD,HESSIAN] = ...

````

Most likely, the solver has tried to start in the initial guess \(x = 0\), and when the objective has been evaluated in a callback to YALMIP, it has led to division by zero.

If we want the solver to start in \(x = 0.5\) instead, we assign that value to the variable, and use the **usex0** option.

````matlab
assign(x,0.5);
ops = sdpsettings('usex0',1);
optimize(Constraints,Objective,ops)

                                Norm of      First-order 
 Iteration        f(x)          step          optimality   CG-iterations
     0                  2                             2                
     1                1.5       0.235702           0.75           1
     2                1.2       0.288675           0.24           1
     3               1.05       0.291606         0.0525           1
     4            1.00435        0.19838        0.00437           1
     5            1.00004       0.065228       3.73e-05           1
     6                  1     0.00610847       2.79e-09           1

Local minimum found.
````

Note, if we restart the solver now, the value assign to the variables are simply those obtained in the latest solve. Hence, calling [optimize](/command/optimize) again will have the solver restart where it ended.


````matlab
optimize(Constraints,Objective,ops)

                                Norm of      First-order 
 Iteration        f(x)          step          optimality   CG-iterations
     0                  1                      2.79e-09                

Initial point is a local minimum.
````

This strategy can be used in cases where you do not have any good guess, but you can solve another problem to find it. Consider the following problem with a simple polytopic constraint, but a complicating objective which has a singularity in \(0\) where   [fmincon](/solver/fmincon) could try to start if we are unlucky.

````matlab
x = sdpvar(3,1);
A = randn(10,3);
b = rand(10,1);
Constraint = [A*x <= b, x>=0];
Objective = -sum(log(x));
````

First solve a feasibility problem to find a solution (we could of course use some simple objective to steer it towards some value, but for simplicity, let us just try to find a solution and hope that it is not \(0\))

````matlab
optimize(Constraint);
````

Now use that solution as an initial guess when solving the nonlinear program

````matlab
 optimize(Constraint,Objective, sdpsettings('usex0',1));
````

Note that it is not necessary that we have the same constraints in the two problems. The only important thing is that we obtain a solution for all of our variables, and that this solution is feasible in the problem we intend to solve.


## Integer programming

In constrast to standard linear and conic programming problems, with integer variables we might benefit from having initial guesses. At the moment, only [gurobi](/solver/gurobi) and [cplex](/solver/cplex) can be supplied initial guesses from YALMIP.

Let us create a random integer program

````matlab
x = sdpvar(10,1);
z = intvar(10,1);
A = randn(60,10);
B = randn(60,10);
b = 200*rand(60,1);
Constraint = [A*(x-1) + B*(z-2) <= b];
Objective = sum(x) + sum(z);
````

Just as above, we use assign to initialize the values (note that we created a problem where \(x = 2\) and \(z=3\) are feasible almost surely)

````matlab
assign(x,2);
assign(z,3);
optimize(Constraint,Objective, sdpsettings('usex0',1));
````

In some cases, we might want to initialize only a partial number of variables. At the moment, only  [gurobi](/solver/gurobi) supports this

````matlab
x = sdpvar(10,1);
z = intvar(10,1);
Constraint = [A*(x-1) + B*(z-2) <= b];
Objective = sum(x) + sum(z);
assign(z,2);
optimize(Constraint,Objective, sdpsettings('usex0',1));
````

### Supplying initials guesses in integer programs might not help much

Important to understand is that initializations of integer programs might have no impact at all. Integer programs can be hard due to two main reasons. The first and perhaps what many think is the hard part, is to find a feasible solution, and then finding the optimal solution. This is called the upper bound generation. The second part is to derive lower bounds on the acheivable performance through relaxations. For some models, it is trivial to find the optimal solution, but it is extremely hard to create good lower bounds and thus proving that the currently best solution actually is the globally optimal solution. In those cases, supplying an initial guess does not help much, as the solver would have found it anyways.

## Hidden variables

When you create high-level models in YALMIP, you only see the variables you explicitly define. However, behind many functions and operators, YALMIP will introduce additional variables to create, e.g., [epigraph reformulations], or to normalize expressions inside nonlinear operators to simplify convexity propagation and computations of derivatives etc. Hence, you assign variables and try to warm-start the solver, but it still fails, as not all variables have an iniital assignment and the solver picks it own initial starting-point instead.

YALMIP tries as far as possible to propagate you initial guesses to the internally introduced variables, but it is not guaranteed to always do this. As an example, the following model uses the linear-programming representable [sumk](command/sumk) operator, and the initial guess on \(x\) will not be propagated to the auxilliary variables required to represent the epigraph of this operator.

````matlab
x = sdpvar(3,1);
Constraint = [sumk(x,2) <= 1];
Objective = sum(1./x);
assign(x,.5);
optimize(Constraint,Objective,sdpsettings('usex0',1))
Your initial point x0 is not between bounds lb and ub; FMINCON
shifted x0 to strictly satisfy the bounds.

                                            First-order      Norm of
 Iter F-count            f(x)  Feasibility   optimality         step
    0       1    6.000000e+00    2.960e+00    2.960e+00
    1       2    3.837333e+00    1.327e+00    1.530e+00    1.467e+00
    2       3    4.358546e+00    5.527e-01    1.231e+00    9.180e-01
    3       5    5.252004e+00    2.841e-01    6.467e-01    2.959e-01
    4       6    6.873002e+00    0.000e+00    1.069e+00    2.779e-01
    5       7    6.676911e+00    0.000e+00    8.641e-02    3.436e-02
    6       8    6.126525e+00    0.000e+00    1.473e-01    1.038e-01
    7       9    6.003090e+00    0.000e+00    1.910e-02    2.588e-02
    8      10    6.001590e+00    0.000e+00    9.931e-05    3.234e-04
    9      11    6.000008e+00    0.000e+00    7.055e-06    3.395e-04
   10      12    6.000003e+00    0.000e+00    2.000e-07    1.019e-06

Local minimum found that satisfies the constraints.
````

For variables which haven't been assigned any values, YALMIP will use the value 0 if the option to use initial guesses is turned on. As we can see in the display, [fmincon](/solver/fmincon) sees that the supplied iniital guess is infeasible, and tweaks it and tries from another point instead. Luckily, it manages to find an alternative inital point which appears to keep our assigned values (inital cost is 6.0, which is the objective for \(x = 0.5\)).


