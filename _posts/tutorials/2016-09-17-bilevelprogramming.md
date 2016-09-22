---
title: "Bilevel programming"
category: tutorial
author_profile: false
level: 4
tags: [Bilevel programming]
excerpt: "Bilevel programming using the built-in bilevel solver"
layout: single
sidebar:
  nav: "tutorials"
---



YALMIP has built-in support for definition, setup, and [solution of bilevel programming] problems. The code here concentrates on the built-in solver for bilevel problems. You can of course set them up yourself, by manually deriving the KKT conditions and solving them using various techniques in YALMIP, or by using YALMIPs high-level [kkt](/command/kkt) operator, as illustrated in the [bilevel example](/example/bilevelalternatives).

For an introduction to bilevel optimization, see [Bard 1999](/reference/bard1999).

### KKT conditions in bilevel programming

The class of bilevel problems that can be adressed natively by YALMIP has to have the following leader-follower (outer-inner) structure

$$
\begin{aligned}
\text{minimize } & f(x,y^{\star})\\
\text{subject to } & (x,y^{\star}) \in \mathcal{C}\\
&\begin{aligned}
y^{\star} = & \arg \min \frac{1}{2}\begin{bmatrix}x\\y\end{bmatrix}^T\begin{bmatrix}H_1 & H_2\\H_2^T & H_3\end{bmatrix}\begin{bmatrix}x\\y\end{bmatrix} + \begin{bmatrix}e_1^T&e_2^T\end{bmatrix}\begin{bmatrix}x\\y\end{bmatrix}\\
& \text{subject to } \, \begin{bmatrix}F_1 & F_2\end{bmatrix}\begin{bmatrix}x\\y\end{bmatrix}\leq h
\end{aligned}
\end{aligned}
$$

The inner problem constraining the follower \\(y\\), is limited to convex quadratic programming problems. The outer problem is allowed to be essentially anything that YALMIP can handle.

The bilevel solver that is available in YALMIP replaces the optimality condition on \\(y^{\star}\\) with the KKT conditions.

$$
\begin{aligned}
H_3y + H_2^Tx + e_2 - F_2^T\lambda &=0\\
\lambda &\geq 0\\
h-F_1x-F_2y & \geq 0\\
\lambda^T(h-F_1x-F_2y) & = 0
\end{aligned}
$$

This is precisely what is done in the manually derived bilevel solution methods in [bilevel example](/example/bilevelalternatives), but the possible benefit of using YALMIPs native support as we will do here is that this solver branches directly on the complementarity conditions, and thus avoids to introduce any numerically dangerous [big-M](/tutorial/bigmandconvexhulls) constants.

### Bilevel linear and quadratic programming

Let us start with a simple bilevel linear programming problem. Start by defining the outer (leader) variables \\(x\\) and inner (follower) variables \\(y\\).

````matlab
sdpvar x1 x2
sdpvar y1 y2 y3
````

Using standard notation, we define the outer constraints and objective

````matlab
OO = -8*x1 - 4*x2 + 4*y1 - 40*y2 + 4*y3;
CO = [y1 y2 y3]>=0,[x1 x2]>=0];
````

followed by the inner constraints and objective

````matlab
OI = x1 + 2*x2 + y1 + y2 + 2*y3;
CI = [-y1 + y2 + y3 <= 1,
      2*x1 - y1 + 2*y2 - 0.5*y3 <= 1,
      2*x2 + 2*y1 - y2 - 0.5*y3 <= 1,
                     [y1 y2 y3] >= 0,
                        [x1 x2] >= 0];
````

Having defined the optimization structures, we call the bilevel solver with the constraints and objectives, and the variables \\(y\\) in order to tell the solver which variables are the inner variables.

````matlab
solvebilevel(CO,OO,CI,OI,[y1 y2 y3]);

* Starting YALMIP bilevel solver.
* Outer solver   : GUROBI-GUROBI
* Inner solver   : GUROBI-GUROBI
* Max iterations : 300
 Node       Upper       Gap(%)      Lower    Open
    1 :          Inf      Inf     -5.000E+01   2  Solved to optimality
    2 :          Inf      Inf     -5.000E+01   3  Solved to optimality
    3 :   -6.000E+00    78.57     -5.000E+01   2  Solved to optimality
    4 :   -6.000E+00    78.57     -5.000E+01   1  Infeasible
    5 :   -6.000E+00    62.50     -2.600E+01   2  Solved to optimality
    6 :   -2.600E+01     0.00     -2.600E+01   0  Solved to optimality
````

As you hopefully have noticed, completely standard YALMIP code is used to setup and manipulate the model. Hence, to obtain the final solution, we use [value](/command/value).

````matlab
value([y1 y2 y3])
ans =
         0    0.6000    0.4000
````

Adding additional complicating constraints is allowed, as long as YALMIP can identify a solver which is capable of solving the outer problem appended with the KKT conditions, excluding the nonconvex complementary slackness constraint. Hence, we can add integrality constraints easily to our model

````matlab
solvebilevel([CO, integer(y2)],OO,CI,OI,[y1 y2 y3]);
````

In the same sense, convex quadratic problems are dealt with straightforwardly

````matlab
solvebilevel(CO,OO+OO^2,CI,OI^2,[y1 y2 y3]);
````


### Bilevel programming with general outer problem

A strong feature of the built-in solver is that it builds upon the infrastructure in YALMIP, and easily hooks up to almost any kind of outer problem. Hence, we can take the problem above, and append a semidefinite constraint to the outer problem. The only difference is that during the branching of the complementary conditions, semidefinite programs have to be solved in each node.

````matlab
CO = [y1 y2 y3] >= 0,[x1 x2] >= 0];
CO = [CO, [1 x1+x2;x1+x2 1/2] >= 0];

solvebilevel(CO,OO,CI,OI,[y1 y2 y3]);
* Starting YALMIP bilevel solver.
* Outer solver   : SeDuMi-1.1
* Inner solver   : GLPK-GLPKMEX
* Max iterations : 300
 Node       Upper       Gap(%)      Lower    Open
    1 :   3.254E-015   100.00    -5.000E+001   2  Solved to optimality
    2 :  -1.723E-010   100.00    -5.000E+001   3  Solved to optimality
    3 :  -4.828E+000    82.39    -5.000E+001   2  Solved to optimality
    4 :  -4.828E+000    82.39    -5.000E+001   1  Infeasible in solver
    5 :  -1.940E+001    13.07    -2.523E+001   2  Solved to optimality
    6 :  -1.940E+001    13.07    -2.523E+001   3  Solved to optimality
    7 :  -1.940E+001    13.07    -2.523E+001   4  Solved to optimality
    8 :  -1.940E+001    13.07    -2.523E+001   3  Infeasible in solver
    9 :  -1.940E+001     7.18    -2.240E+001   4  Solved to optimality
   10 :  -1.940E+001     7.18    -2.240E+001   3  Infeasible in solver
   11 :  -1.940E+001     0.00    -1.940E+001   2  Infeasible in solver
````

The bilevel solver is restricted to convex quadratic inner problems, but convexity is not a requirement on the outer problems. The bilevel solver solves the outer problem repeatedly in a branch-and-bound procedure, with additional equality constraints derived from complementary slackness appended. Hence, we have to make a choice between global solution of the outer problem, or a simple local solution. If we go for a local solution (using, e.g, [FMINCON]), we have no guarantees (except that the inner optimality constraint is satisfied). If we use a global outer solver (such as [BMIBNB]), a globally optimal bilevel solution follows.

As an illustration, we solve the original problem, but append a nonconvex quadratic term to the outer problem. To ensure a globally optimal solution, we use the global solver [BMIBNB] as the outer solver.

````matlab
OO = -8*x1 - 4*x2 + 4*y1 - 40*y2 + 4*y3;
CO = [[y1 y2 y3] >= 0,[x1 x2] >= 0];
OI = x1 + 2*x2 + y1 + y2 + 2*y3;
CI = [-y1 + y2 + y3 <= 1,
      2*x1 - y1 + 2*y2 - 0.5*y3 <= 1,
      2*x2 + 2*y1 - y2 - 0.5*y3 <= 1,
                     [y1 y2 y3] >= 0,
                        [x1 x2] >= 0];
ops = sdpsettings('bilevel.outersolver','bmibnb');
solvebilevel(CO,OO-OO^2,CI,OI,[y1 y2 y3],ops)
* Starting YALMIP bilevel solver.
* Outer solver   : BMIBNB
* Inner solver   : GLPK-GLPKMEX
* Max iterations : 300
 Node       Upper       Gap(%)      Lower    Open
    1 :  -1.110E-015   100.00    -2.550E+003   2  Solved to optimality
    2 :  -1.110E-015   100.00    -2.550E+003   3  Solved to optimality
    3 :  -1.110E-015   100.00    -2.550E+003   4  Solved to optimality
    4 :  -1.110E-015   100.00    -2.550E+003   5  Solved to optimality
    5 :  -7.020E+002    56.83    -2.550E+003   4  Solved to optimality
    6 :  -7.020E+002    56.83    -2.550E+003   3  Infeasible in solver
    7 :  -7.020E+002    30.97    -1.332E+003   2  Solved to optimality
    8 :  -7.020E+002    30.97    -1.332E+003   3  Solved to optimality
    9 :  -7.020E+002    30.97    -1.332E+003   2  Solved to optimality
   10 :  -7.020E+002    30.97    -1.332E+003   1  Infeasible in solver
   11 :  -7.020E+002     0.00    -7.020E+002   0  Solved to optimality
````
