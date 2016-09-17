---
title: "Moment relaxations"
type: tutorial
author_profile: false
level: 4
tags: [Semidefinite programming, Moment relaxations]
excerpt: "Moment relaxations allows us to find lower bounds on polynomial optimization problems using semidefinite programming"
layout: single
sidebar:
  nav: "tutorials"
---


YALMIP comes with a built-in module for polynomial programming using moment relaxations. This can be used for finding lower bounds on constrained polynomial programs (inequalities and equalities, element-wise and semidefinite), and to extract the optimizers in case the relaxation is tight. The implementation is entirely based on high-level YALMIP code, and can be somewhat inefficient for large problems (the inefficiency would then show in the setup of the problem, not actually solving the semidefinite resulting program). For the underlying theory of moment relaxations, the reader is referred to [Lasserre 2001].

### Solving polynomial problems by relaxations

The following code calculates a lower bound on a concave quadratic optimization problem. As you can see, the only difference compared to solving the problem using a standard solver, such as [FMINCON] or [SNOPT], or the global solver [BMIBNB], is that we call [solvemoment] instead of [optimize] (an alternative is to call [optimize] and specify `'moment'` as the solver in [sdpsettings]).

````matlab
sdpvar x1 x2 x3
obj = -2*x1+x2-x3;
F = [x1*(4*x1-4*x2+4*x3-20)+x2*(2*x2-2*x3+9)+x3*(2*x3-13)+24>=0;
                  4-(x1+x2+x3)>=0;
                    6-(3*x2+x3)>=0;
     2>=x1>=0,x2>=0,3>=x3>=0]
solvemoment(F,obj);
value(obj)

 ans =
     -6.0000
````

Notice that YALMIP does not recover variables by default, a fact showing up in the difference between lifted variables and actual nonlinear variables (lifted variables are the variables used in the semidefinite relaxation to model nonlinear variables). The linear variables coincide with the relaxed linear variables, hence the relaxed value of the linear objective can be checked directly through [value]. The lifted variables can be obtained by using the command [relaxvalue]. The quadratic constraint above is satisfied in the lifted variables, but not in the true variables, as the following code illustrates.

````matlab
relaxvalue(x1*(4*x1-4*x2+4*x3-20)+x2*(2*x2-2*x3+9)+x3*(2*x3-13)+24)

 ans =

   23.2648
value(x1*(4*x1-4*x2+4*x3-20)+x2*(2*x2-2*x3+9)+x3*(2*x3-13)+24)

 ans =

  -2.0000
````

A tighter relaxation can be obtained by using a higher order relaxation (the lowest possible is used if it is not specified).

````matlab
solvemoment(F,obj,[],2);
value(obj)

 ans =
     -5.6593
````

On this particular problem, the obtained bound can be used iteratively to improve the bound by adding dynamically generated cuts.

````matlab
solvemoment([F, obj>=value(obj)],obj,[],2);
value(obj)

 ans =
     -5.3870
solvemoment([F, obj>=value(obj)],obj,[],2);
value(obj)

 ans =

     -5.1270
````

The known true minima, -4, is found in the fourth order relaxation.

````matlab
solvemoment(F,obj,[],4);
value(obj)

 ans =
     -4.0000
````

The true global minima is however not recovered with the lifted variables, as we can see if we check the current solution (still violates the nonlinear constraint).

````matlab
check(F)

+++++++++++++++++++++++++++++++++++++++++++++++++++
| ID| Constraint   |         Type| Primal residual|
+++++++++++++++++++++++++++++++++++++++++++++++++++
| #1| Numeric value| Element-wise|        -0.88573|
| #2| Numeric value| Element-wise|           1.834|
| #3| Numeric value| Element-wise|           5.668|
| #4| Numeric value| Element-wise|           1.834|
| #5| Numeric value| Element-wise|         0.16599|
| #6| Numeric value| Element-wise|     2.0873e-006|
| #7| Numeric value| Element-wise|         0.33198|
| #8| Numeric value| Element-wise|           2.668|
+++++++++++++++++++++++++++++++++++++++++++++++++++
````

### Extracting solutions

To extract a (or several) globally optimal solution, we need two output arguments. The first output is a diagnostic structure (standard solution structure from the semidefinite solver), the second output is the (hopefully, it is not always to extract solutions, even though the bound is tight) extracted globally optimal solutions and the third output is a data structure containing all data that was needed to extract the solution.

````matlab
[sol,x,momentdata] = solvemoment(F,obj,[],4);
x{1}

ans =

    0.5000
    0.0000
    3.0000
x{2}

ans =

    2.0000
   -0.0000
   -0.0000
````

Assigning any of the extracted solutions should yield a feasible set of constraint, and an objective which achieves the lower bound -4 that we just computed. Too avoid any confusion about the ordering of variables, we use the third output to make sure we assign the solution to the correct variables.

````matlab
assign(momentdata.x,x{1});
check(F)
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
|   ID|      Constraint|                                 Type|   Primal residual|
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
|   #1|   Numeric value|   Elementwise inequality (quadratic)|      -4.1666e-006|
|   #2|   Numeric value|               Elementwise inequality|               0.5|
|   #3|   Numeric value|               Elementwise inequality|                 3|
|   #4|   Numeric value|               Elementwise inequality|               0.5|
|   #5|   Numeric value|               Elementwise inequality|               1.5|
|   #6|   Numeric value|               Elementwise inequality|       5.9515e-007|
|   #7|   Numeric value|               Elementwise inequality|                 3|
|   #8|   Numeric value|               Elementwise inequality|       1.4818e-006|
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
````


Since we know in what order the variables were defined, we could have used the following version too

````matlab
assign([x1;x2;x3],x{1});

value(obj)
ans =

   -4.0000
````
