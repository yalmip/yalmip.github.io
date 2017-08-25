
---
layout: single
permalink: /Initials
excerpt: "Give your solver a hint"
title: "Supplying initial guesses to solvers"
tags: []
comments: true
date: '2017-08-25'
---

In some cases, the solver might benefit from an initial guess on where to start the search. 

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

In constrast to standard linear and conic programming, with integer variables we might benefit from having initial guesses. At the moment, only [gurobi](/solver/gurobi) and [cplex](/solver/cplex) can be supplied initial guesses from YALMIP.

Let us create a random integer program

````matlab
x = sdpvar(10,1);
z = intvar(10,1);
A = randn(60,10);
B = randn(60,10);
b = 200*rand(60,1);
Constraint = [A*x + B*z <= b];
Objective = sum(x) + sum(z);
optimize(Constraint,Objective)
````








