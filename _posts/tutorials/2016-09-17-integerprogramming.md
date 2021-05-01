---
title: "Integer programming"
category: tutorial
level: 4
tags: [Integer programming]
excerpt: "Undisciplined programming often leads to integer models, but in some cases you have no option."
sidebar:
  nav: "tutorials"
---


YALMIP supports several [mixed integer programming solvers], but also comes with a very simple [built-in solver for mixed integer programming (BNB)](/solver/bnb) (which you shouldn't use unless you absolutely have to), based on a simple standard branch-and-bound algorithm.

### Integer and binary variables

Defining binary and integer variables is done with the commands [binvar](/command/binvar) and [intvar](/command/intvar). The resulting objects are essentially [sdpvar](/command/sdpvar) objects with implicit constraints.

````matlab
x = intvar(n,m);
y = binvar(n,m);
````

Objects with integer variables are manipulated as standard [sdpvar](/command/sdpvar) objects.

````matlab
z = x + y + trace(x) + sum(sum(y));
F = [z >= 0, x <= 0];
````

In the code above, integrality was imposed by using integer and binary variables. An equivalent alternative is to use explicit constraints.

````matlab
x = sdpvar(n,m);
y = sdpvar(n,m);
z = x + y + trace(x) + sum(sum(y));
F = [z >= 0, x <= 0, integer(x), binary(y)];
````


### Mixed integer conic programming

The global integer solver can be applied to any kind of conic program that can be defined within the YALMIP framework, and defining integer programs is as simple as defining standard problems. In addition to the external supported mixed integer solvers, YALMIP comes with an internal branch-and-bound solver, called [BNB], to be used together with any continuous solver. Hence, it is possible to solve mixed integer linear/quadratic/second order cone/semidefinite/geometric programs in YALMIP. Note that the internal branch-and-bound algorithm is rudimentary and useful only for small problems.

As an example, let us return to the [linear regression problem](/tutorial/linearprogramming). Solving the same problems, but looking for integer solutions can be done by changing one line of code

````matlab
x = [1 2 3 4 5 6]';
t = (0:0.02:2*pi)';
a = [sin(t) sin(2*t) sin(3*t) sin(4*t) sin(5*t) sin(6*t)];
y = a*x+(-4+8*rand(length(a),1));

x_hat = intvar(6,1);

residuals = y-a*x_hat;
bound = sdpvar(length(residuals),1);
F = [-bound <= residuals <= bound];
optimize(F,sum(bound));
x_L1 = value(x_hat);
optimize([],residuals'*residuals);
x_L2 = value(x_hat);
bound = sdpvar(1,1);
F = [-bound <= residuals <= bound];
optimize(F,bound);
x_Linf = value(x_hat);
````

Since [BNB](/solver/bnb) supports mixed integer semidefinite programming, we can easily solve the problems above with semidefinite constraints.

````matlab
F = [toeplitz(x_hat) > 0];
optimize(F,residuals'*residuals);
x_L2_toep = value(x_hat);
````

Note that [BNB](/solver/bnb)  not should be used if you have simple mixed integer linear programs. In that case, you can just as well download a much faster free specialized MILP solver, such as [GLPK](solver/glpk) or academic license version of [GUROBI](/solver/gurobi).

### General mixed integer programming

The mixed integer programming solvers discussed above are all guaranteed to find a globally optimal solution, if one exists. The built-in branch-and-bound module can be applied also to general nonlinear programs with discrete data. The difference is that there is no guarantee on global optimality for these problems. It can however be a useful strategy for finding reasonably good feasible solutions to mixed integer nonlinear programs.

````matlab
x = sdpvar(5,1);
A = randn(15,5);
b = rand(15,1)*10;

obj = sum(x) + sum((x-3).^4); % 4th order objective
ops = sdpsettings('solver','bnb','bnb.solver','fmincon');
optimize([A*x <= b, integer(x)],obj,ops)
* Starting YALMIP integer branch & bound.
* Lower solver   : fmincon-standard
* Upper solver   : rounder
* Max iterations : 300

Warning : The relaxed problem may be nonconvex. This means
that the branching process not is guaranteed to find a
globally optimal solution, since the lower bound can be
invalid. Hence, do not trust the bound or the gap...
 Node       Upper       Gap(%)      Lower    Open
    1 :   1.090E+002    22.14     6.948E+001   2  
    2 :   1.090E+002    22.14     6.948E+001   3  
    3 :   1.090E+002     6.24     9.619E+001   2  
    4 :   1.090E+002     6.24     9.619E+001   3  
    5 :   1.090E+002     5.98     9.669E+001   4  
    6 :   1.090E+002     5.98     9.669E+001   3  
    7 :   1.070E+002     4.81     9.718E+001   4  
    8 :   1.070E+002     4.81     9.718E+001   5  
    9 :   1.070E+002     3.91     9.895E+001   4  
   10 :   1.070E+002     3.91     9.895E+001   3  
   11 :   1.070E+002     0.64     1.056E+002   2  
   12 :   1.070E+002     0.64     1.056E+002   3  
   13 :   1.070E+002     0.43     1.061E+002   4  
   14 :   1.070E+002     0.43     1.061E+002   5  
   15 :   1.070E+002     0.39     1.062E+002   6  
   16 :   1.070E+002     0.39     1.062E+002   7  
   17 :   1.070E+002     0.22     1.065E+002   6  
   18 :   1.070E+002     0.22     1.065E+002   5  
   19 :   1.070E+002     0.19     1.066E+002   4  
   20 :   1.070E+002     0.19     1.066E+002   3  
   21 :   1.070E+002     0.18     1.066E+002   2  
   22 :   1.070E+002     0.18     1.066E+002   1  
   23 :   1.070E+002     0.18     1.066E+002   0  
+  23 Finishing.  Cost: 107
````

If globality is desired, and the problem is nonconvex (for relaxed integers), the [global optimization](/tutorial/globaloptimization) solver [BMIBNB](/solver/bmibnb) does actually support integer variables. However, [BMIBNB](/solver/bmibnb) is only intended for a few number of variables, and is much slower, since it performs branching not only in the integer variables, but also in the continuous variables.

````matlab
ops = sdpsettings('solver','bmibnb','bmibnb.upper','fmincon');
optimize([A*x <= b, integer(x)],obj,ops)
* Starting YALMIP bilinear branch & bound.
* Upper solver   : fmincon
* Lower solver   : CPLEX
* LP solver      : GLPK
 Node       Upper      Gap(%)       Lower    Open
    1 :   1.090E+002  15378719.56    -1.692E+007   2  Improved solution  
    2 :   1.090E+002  15378719.56    -1.692E+007   1  Infeasible  
    3 :   1.090E+002  867849.97    -9.545E+005   2    
    4 :   1.090E+002  867849.97    -9.545E+005   1  Infeasible  
    5 :   1.090E+002  142704.35    -1.569E+005   2    
    6 :   1.090E+002  142704.35    -1.569E+005   1  Infeasible  
    7 :   1.090E+002  36150.61    -3.966E+004   2    
    8 :   1.090E+002  36150.61    -3.966E+004   1  Poor bound  
    9 :   1.090E+002  13832.35    -1.511E+004   2    
   10 :   1.090E+002  13832.35    -1.511E+004   3    
   11 :   1.090E+002  6307.24    -6.829E+003   4    
   12 :   1.090E+002  6307.24    -6.829E+003   5    
   13 :   1.090E+002  3897.54    -4.178E+003   6    
...
   94 :   1.070E+002     8.66     9.765E+001   1  Infeasible  
   95 :   1.070E+002     0.00     1.210E+002   0  Poor bound  
+ 95 Finishing.  Cost: 107 Gap: 0%
````

For an additional example, check out the [mixed integer geometric programming example](/tutorial/geometricprogramming).
