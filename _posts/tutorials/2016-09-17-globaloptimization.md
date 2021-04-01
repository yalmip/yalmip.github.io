---
title: "Global optimization"
category: tutorial
author_profile: false
level: 4
tags: [Nonconvex programming, Global optimization, Nonconvex quadratic programming, Nonlinear semidefinite programming, BMI, Bilinear matrix inequality]
excerpt: "The holy grail! 60% of the time it works every time."
layout: single
published: true
sidebar:
  nav: "tutorials"
---

Global solutions! Well, don't expect too much from global solvers. The problem we attack here is intractable in theory almost always, and practice is not far from theory unfortunately.

The focus here is on the built-in solver [BMIBNB](/solver/bmibnb). The solver is fairly robust on small problems and is probably the most general nonlinear global solver available, albeit not the fastest. External global solvers interfaced can be found [here](/tags/#global-solver)

The [BMIBNB](/solver/bmibnb) solver is based on a simple spatial branch & bound strategy using convex envelope approximations for nonlinear operators. LP-based bound tightening is (optionally) applied iteratively to improve variable bounds together with a large array of additional techniques to, e.g., exploit complementary constraints, extract bounds from quadratic constraints and objective bounds, exploiting inverse functions, strengthening using semidefinite relaxations etc. See the tutorial [envelope approximation](/tutorial/envelopesinbmibnb) for some of the details.

Relaxed problems are solved using either an [LP solver](/tags/#linear-programming-solver), [QP solver](/tags/#quadratic-programming-solver), [SOCP solver](/tags/#second-order-cone-programming-solver), or [SDP solver](/tags/#semidefinite-programming-solver) solver, depending on the problem and what solvers you have available, while upper bounds are found using a [local nonlinear solver](/tags/#nonlinear-programming-solver) such as [FMINCON](/solver/fmincon),  [SNOPT](/solver/snopt) or [IPOPT](/solver/ipopt).

### Nonconvex quadratic programming
The first example is a problem with a concave quadratic constraint (this is the example addressed in the [moment relaxation tutorial](/tutorial/momentrelaxations)). Three different optimization problems are solved during the branching: Upper bounds using a local nonlinear solver **'bmibnb.uppersolver'**, lower bounds with **'bmibnb.lowersolver'** and bound tightening using a linear programming solver **'bmibnb.lpsolver'**. All these solvers are selected automatically, but you can change them of course.

````matlab
clear all
x1 = sdpvar(1,1);
x2 = sdpvar(1,1);
x3 = sdpvar(1,1);
p = -2*x1+x2-x3;
F = [x1*(4*x1-4*x2+4*x3-20)+x2*(2*x2-2*x3+9)+x3*(2*x3-13)+24>=0,
     4-(x1+x2+x3)>=0,
      6-(3*x2+x3)>=0,
               x1>=0,
             2-x1>=0,
               x2>=0,
               x3>=0,
             3-x3>=0]

options = sdpsettings('solver','bmibnb');
optimize(F,p,options);
* Starting YALMIP global branch & bound.
* Upper solver     : fmincon
* Lower solver     : GUROBI
* LP solver        : GUROBI
* -Extracting bounds from model
* -Perfoming root-node bound propagation
* -Feasible solution found by heuristics
* -Calling upper solver (found a solution!)
* -Branch-variables : 3
* -More root-node bound-propagation
* -Performing LP-based bound-propagation 
* -And some more root-node bound-propagation
* Starting the b&b process
 Node       Upper       Gap(%)       Lower     Open   Time
    1 :  -4.00000E+00    33.33   -6.00000E+00    2     0s    
    2 :  -4.00000E+00    33.33   -6.00000E+00    3     0s    
    3 :  -4.00000E+00    28.01   -5.62857E+00    4     0s    
    4 :  -4.00000E+00    28.01   -5.62857E+00    3     0s  Poor bound in lower, killing node  
    5 :  -4.00000E+00    13.56   -4.72758E+00    2     0s  Infeasible in node bound-propagation  
    6 :  -4.00000E+00    13.56   -4.72758E+00    1     0s  Infeasible in node bound-propagation  
    7 :  -4.00000E+00     0.00   -4.00000E+00    0     0s  Poor bound in lower, killing node  
* Finished.  Cost: -4 (lower bound: -4, relative gap 0%)
* Termination with all nodes pruned 
* Timing: 26% spent in upper solver (4 problems solved)
*         3% spent in lower solver (5 problems solved)
*         39% spent in LP-based domain reduction (112 problems solved)
*         2% spent in upper heuristics (80 candidates tried)
````

As you can see, a lot of time is spent in the linear programming based bound propagation. [BMIBNB](solver/bmibnb) applies this on almost all models. If you want to turn it off, you set **'bmibnb.lpreduce'** to **0**, and if you want to enforce it you set it to **1**. Default is **-1** which means [BMIBNB](solver/bmibnb) decides.

Upper bounds are generated both by calling the nonlinear solver, but also by aggresively checking all kinds of candidates that pop up in the algorithm (solutions to lower bound problems, and points created when performing bound propagation). In the log above, we see that the nonlinear solver was only called 4 times, but 80 other solution candidates were analyzed as candidates for generating an upper bound. Also note that the globally optimal solution was solution was available already in the first node, so all effort was effectively spent on certifying optimality. On some models finding the optmal solution is hard, and on some models the certification is the hard part, and you typically don't know which it is until you have solved the problem.

The second example is a slightly larger nonconvex quadratic programming problem. The problem is immediately solved to a gap of less than 1%.

````matlab
clear all
x1 = sdpvar(1,1);
x2 = sdpvar(1,1);
x3 = sdpvar(1,1);
x4 = sdpvar(1,1);
x5 = sdpvar(1,1);
x6 = sdpvar(1,1);
p = -25*(x1-2)^2-(x2-2)^2-(x3-1)^2-(x4-4)^2-(x5-1)^2-(x6-4)^2;
F = [(x3-3)^2+x4>=4,(x5-3)^2+x6>=4,x1-3*x2<=2, -x1+x2<=2,
     x1-3*x2<=2, x1+x2>=2,6>=x1+x2>=2,1<=x3<=5, 0<=x4<=6, 1<=x5<=5, 0<=x6<=10, x1>=0,x2>=0];

options = sdpsettings('solver','bmibnb');
optimize(F,p,options);
* Starting YALMIP global branch & bound.
* Upper solver     : fmincon
* Lower solver     : GUROBI
* LP solver        : GUROBI
* -Extracting bounds from model
* -Perfoming root-node bound propagation
* -Calling upper solver (found a solution!)
* -Branch-variables : 6
* -More root-node bound-propagation
* -Performing LP-based bound-propagation 
* -And some more root-node bound-propagation
* Starting the b&b process
 Node       Upper       Gap(%)       Lower     Open   Time
    1 :  -3.10000E+02     0.96   -3.13000E+02    2     1s  Improved solution found by heuristics  
* Finished.  Cost: -310 (lower bound: -313, relative gap 0.96%)
* Termination with relative gap satisfied 
* Timing: 42% spent in upper solver (2 problems solved)
*         2% spent in lower solver (1 problems solved)
*         18% spent in LP-based domain reduction (24 problems solved)
*         2% spent in upper heuristics (9 candidates tried)
````

However, there is a gap here which we might want to close. The solution has an objective -310 while the lower bound is at -313 (which is within the default tolerance for terminating). Tightening the tolerances allows us to proceed further, and this reveals that the solution found in the root node indeed was the global solution and the culprit for the gap was the lower bound.

````matlab
options = sdpsettings('solver','bmibnb','bmibnb.relgaptol',1e-4);
optimize(F,p,options);
* Starting YALMIP global branch & bound.
* Upper solver     : fmincon
* Lower solver     : GUROBI
* LP solver        : GUROBI
* -Extracting bounds from model
* -Perfoming root-node bound propagation
* -Calling upper solver (found a solution!)
* -Branch-variables : 6
* -More root-node bound-propagation
* -Performing LP-based bound-propagation 
* -And some more root-node bound-propagation
* Starting the b&b process
 Node       Upper       Gap(%)       Lower     Open   Time
    1 :  -3.10000E+02     0.96   -3.13000E+02    2     0s  Improved solution found by heuristics  
    2 :  -3.10000E+02     0.96   -3.13000E+02    1     0s  Infeasible in node bound-propagation  
    3 :  -3.10000E+02     0.00   -3.10000E+02    2     0s    
* Finished.  Cost: -310 (lower bound: -310.0001, relative gap 4.5229e-05%)
* Termination with relative gap satisfied 
* Timing: 43% spent in upper solver (3 problems solved)
*         2% spent in lower solver (2 problems solved)
*         34% spent in LP-based domain reduction (69 problems solved)
*         2% spent in upper heuristics (43 candidates tried)
````

### Nonconvex polynomial programming

Polynomial programs are transformed to to bilinear programs. As an example, the variable \\(x^3y^2\\) will be replaced with the the variable \\(w\\) and the constraints \\(w = uv\\), \\(u = zx\\), \\(z = x^2\\), \\(v == y^2\\). This is done automatically if the global solver is called with a higher order polynomial problem. With this transformation, standard bilinear envelopes can be used in the creation of the relaxations for computing lower bounds. Note though, the nnolinear solver will work with the original polynomial model, so the bilinearization is only used for the lower bound computations.

````matlab
sdpvar x y
F = [x^3+y^5 <= 5, y >= 0];
options = sdpsettings('verbose',1,'solver','bmibnb');
optimize(F,-x,options)
* Starting YALMIP global branch & bound.
* Upper solver     : fmincon
* Lower solver     : GUROBI
* LP solver        : GUROBI
* -Extracting bounds from model
* -Perfoming root-node bound propagation
* -Calling upper solver (found a solution!)
* -Branch-variables : 2
* -More root-node bound-propagation
* -Performing LP-based bound-propagation 
* -And some more root-node bound-propagation
* Starting the b&b process
 Node       Upper       Gap(%)       Lower     Open   Time
    1 :  -1.70998E+00     0.00   -1.70998E+00    0     0s  Poor bound in lower, killing node  
* Finished.  Cost: -1.71 (lower bound: -1.71, relative gap 0%)
* Termination with all nodes pruned 
* Timing: 47% spent in upper solver (1 problems solved)
*         2% spent in lower solver (1 problems solved)
*         13% spent in LP-based domain reduction (6 problems solved)
*         1% spent in upper heuristics (2 candidates tried)
````

Remember that spatial branch & bound and envelope approximations rely on bounds on the involved variables, but our model lacks any explicit bound on \\(x\\) and has only a lower bound on \\(y\\), yet [BMIBNB](/solver/bmibnb) does not complain about this. The reason is that [BMIBNB](/solver/bmibnb) continuously tries to extract and improve bounds. Since \\(y\\) is non-negative and \\(x^3 \leq 5-y^3\\) it directly implies \\(x \leq 5^{1/3}\\). Already in the root node, the nonlinear solver finds the solution \\(x=5^{1/3}, y=0\\) (which later is proven optimal). Since this solution defines an upper bound it implies \\(-x \leq -5^(1/3)\\), i.e. \\(5^(1/3)\) is both an upper and lower bound for \\(x)\\. This propagates through \\(y^5 \leq 5-(5^{1/3})^3\\) which is \\(y^5 \leq 0\), i.e., \\(y\leq 0\\) and we have effectively solved the problem by bound propagation driven by the first solution seen. Due to numerical tolerances in the nonlinear solver and trickle-flow through all the nonlinear operations, the bounds will not be exactly tight in presolve, but the gap will be small enough for the solver to terminate in the first node after having computed a lower bound.

### General nonconvex programming

The solver supports global optimization over almost all operators supported on decision variables in YALMIP, as most operators are equipped with [envelope generators](/tutorial/envelopesinbmibnb). Hence, nothing prevents us from, e.g., nonconvex global trigonometric optimization.

````matlab
sdpvar x y 
p = sin(1+y*x)^2+cos(y*x);
optimize([-1 <= [x y] <= 1],p,sdpsettings('solver','bmibnb'));
* Starting YALMIP global branch & bound.
* Upper solver     : fmincon
* Lower solver     : GUROBI
* LP solver        : GUROBI
* -Extracting bounds from model
* -Perfoming root-node bound propagation
* -Calling upper solver (found a solution!)
* -Branch-variables : 3
* -More root-node bound-propagation
* -Performing LP-based bound-propagation 
* -And some more root-node bound-propagation
* Starting the b&b process
 Node       Upper       Gap(%)       Lower     Open   Time
    1 :   5.40302E-01     0.00    5.40302E-01    0     0s  Poor bound in lower, killing node  
* Finished.  Cost: 0.5403 (lower bound: 0.5403, relative gap 0%)
* Termination with all nodes pruned 
* Timing: 14% spent in upper solver (1 problems solved)
*         3% spent in lower solver (1 problems solved)
*         39% spent in LP-based domain reduction (10 problems solved)
*         1% spent in upper heuristics (2 candidates tried)
````

### Nonconvex semidefinite programming

The following problem is a BMI (bilinear matrix inequality) problem which has been used as a test example in some papers on BMIs

````matlab
yalmip('clear')
x = sdpvar(1,1);
y = sdpvar(1,1);
t = sdpvar(1,1);
A0 = [-10 -0.5 -2;-0.5 4.5 0;-2 0 0];
A1 = [9 0.5 0;0.5 0 -3 ; 0 -3 -1];
A2 = [-1.8 -0.1 -0.4;-0.1 1.2 -1;-0.4 -1 0];
K12 = [0 0 2;0 -5.5 3;2 3 0];
F = [x>=-0.5, x<=2, y>=-3, y<=7];
F = [F, A0+x*A1+y*A2+x*y*K12-t*eye(3)<=0];
options = sdpsettings('solver','bmibnb');
optimize(F,t,options);
* Starting YALMIP global branch & bound.
* Upper solver     : fmincon
* Lower solver     : MOSEK
* LP solver        : GUROBI
* -Extracting bounds from model
* -Perfoming root-node bound propagation
* -Feasible solution found by heuristics
* -Calling upper solver (no solution found)
* -Branch-variables : 2
* -More root-node bound-propagation
* -Performing LP-based bound-propagation 
* -And some more root-node bound-propagation
* Starting the b&b process
 Node       Upper       Gap(%)       Lower     Open   Time
    1 :   2.22045E-16    65.14   -9.66147E-01    2     0s  Added 1 cut on SDP cone  
    2 :   2.22045E-16    65.14   -9.66147E-01    3     0s  Added 1 cut on SDP cone  
    3 :   2.22045E-16    64.93   -9.61375E-01    4     0s  Added 1 cut on SDP cone  
    4 :  -9.51670E-01     0.50   -9.61375E-01    3     0s  Improved solution found by heuristics | Added 1 cut on SDP cone | Pruned stack based on new upper bound  
* Finished.  Cost: -0.95167 (lower bound: -0.96137, relative gap 0.49603%)
* Termination with relative gap satisfied 
* Timing: 37% spent in upper solver (5 problems solved)
*         24% spent in lower solver (8 problems solved)
*         16% spent in LP-based domain reduction (42 problems solved)
*         2% spent in upper heuristics (40 candidates tried)
````

The default behaviour to attack BMIs in [BMIBNB](/solver/bmibnb) is by employing a [nonconvex cutting plane strategy for the upper bound generation](/nonlinearglobalsdp). Hence we see the use of a standard nonlinear solver [FMINCON](/solver/bmibnb) for the upper bounds in the log above, despite the fact that the problem involves a semidefinite cone. A linear semidefinite programming solver is required for the lower bounds.

As a second BMI problem, we will solve an linear quadratic (LQ) control problem where we want to find a state-feedback matrix with bounded elements. For the global code to work, global lower and upper and bound on all complicating variables (involved in nonlinear terms) should be supplied, either explicitly or implicitly in the linear constraints. In this example, the variable **K** is already bounded in the original problem formulation (i.e. the problem we want to solve is LQR with a bounded feedback matrix), but the elements in **P** are not, hence the warning about unbounded variables. 

````matlab
yalmip('clear')
A = [-1 2;-3 -4];B = [1;1];
P = sdpvar(2,2);
K = sdpvar(1,2);
F = [P>=0, (A+B*K)'*P+P*(A+B*K) <= -eye(2)-K'*K];
F = [F, -0.1 <= K <= 0.1];
options = sdpsettings('solver','bmibnb');
optimize(F,trace(P),options);
* Starting YALMIP global branch & bound.
* Upper solver     : fmincon
* Lower solver     : MOSEK
* LP solver        : GUROBI
* -Extracting bounds from model
* -Perfoming root-node bound propagation
* -Calling upper solver (no solution found)
* -Branch-variables : 5
* -More root-node bound-propagation
* -Performing LP-based bound-propagation 
* -And some more root-node bound-propagation
* Warning: 1 branch variable is unbounded from below
* Warning: 3 branch variables are unbounded from above
* Starting the b&b process
 Node       Upper       Gap(%)       Lower     Open   Time
    1 :   4.83416E-01    19.23    2.23171E-01    2     0s  Solution found by upper solver  
    2 :   4.83416E-01    19.23    2.23171E-01    3     0s    
    3 :   4.83416E-01     0.01    4.83276E-01    2     0s  Infeasible in node bound-propagation  
* Finished.  Cost: 0.48342 (lower bound: 0.48328, relative gap 0.0094351%)
* Termination with relative gap satisfied 
* Timing: 25% spent in upper solver (3 problems solved)
*         15% spent in lower solver (12 problems solved)
*         40% spent in LP-based domain reduction (45 problems solved)
*         3% spent in upper heuristics (38 candidates tried)
````

THe solver easily derives that the diagonal elements of \\(P\\) have to be non-negative but no upper bounds were found on them in presolve, and it has failed to derive any kind of bounds on the non-diagonal elements. Nevertheless, in this particular case  [BMIBNB](/solver/bmibnb) manages to solve the problem, but lack of bounds can easily lead to failure to converge or even generate sensible lower bounds, so it is adviced to add explicit bounds to all variables, and to give serious thought into reasonable values.

Notice that we have some degree-of-freedom in how we model this problem. The Lyapunov function which is nonlinear in \\(K\\) and \\(P\\) can be partially linearized by applying a Schur complement. Hece, we can test to trade some reduced non-linearity against a larger semidefinite cone. For this particular example, it is not an improvement. The upper bound heuristics (nonlinear cutting planes) struggles to find a good solution, while the lower bound is pretty tight early on. More on how to tweak the cutting plane strategy for the upper bounds are found in the post [cutting plane strategy for the upper bound generation](/nonlinearglobalsdp).

````matlab
F = [P>=0, [-eye(2) - ((A+B*K)'*P+P*(A+B*K)) K';K 1] >= 0];
F = [F, K<=0.1, K>=-0.1];
options = sdpsettings('solver','bmibnb');
optimize(F,trace(P),options);
* Starting YALMIP global branch & bound.
* Upper solver     : fmincon
* Lower solver     : MOSEK
* LP solver        : GUROBI
* -Extracting bounds from model
* -Perfoming root-node bound propagation
* -Calling upper solver (no solution found)
* -Branch-variables : 5
* -More root-node bound-propagation
* -Performing LP-based bound-propagation 
* -And some more root-node bound-propagation
* Warning: 1 branch variable is unbounded from below
* Warning: 3 branch variables are unbounded from above
* Starting the b&b process
 Node       Upper       Gap(%)       Lower     Open   Time
    1 :           Inf      NaN    2.25425E-01    2     0s  Added 1 cut on SDP cone  
    2 :           Inf      NaN    2.25425E-01    3     0s  Added 1 cut on SDP cone  
    3 :   9.85692E-01    30.24    4.64113E-01    4     0s  Solution found by heuristics | Added 1 cut on SDP cone  
    4 :   8.32926E-01    22.37    4.64113E-01    3     0s  Improved solution found by heuristics | Added 1 cut on SDP cone | Pruned stack based on new upper bound  
    5 :   8.32926E-01    21.17    4.82006E-01    2     0s  Added 1 cut on SDP cone | Pruned stack based on new upper bound  
    6 :   8.32926E-01    21.17    4.82006E-01    3     0s  Added 2 cuts on SDP cone  
    7 :   8.32926E-01    21.15    4.82320E-01    4     0s  Added 1 cut on SDP cone  
    8 :   8.32926E-01    21.15    4.82320E-01    5     0s  Added 1 cut on SDP cone  
    9 :   8.32926E-01    21.11    4.82949E-01    6     0s  Added 1 cut on SDP cone  
   10 :   8.32926E-01    21.11    4.82949E-01    7     0s  Added 1 cut on SDP cone  
   11 :   5.35460E-01     3.46    4.83219E-01    5     1s  Improved solution found by heuristics | Added 1 cut on SDP cone | Pruned stack based on new upper bound  
   12 :   5.35460E-01     3.46    4.83219E-01    4     1s  Added 1 cut on SDP cone | Pruned stack based on new upper bound  
   13 :   5.35460E-01     3.45    4.83326E-01    5     1s  Added 1 cut on SDP cone  
   14 :   5.35460E-01     3.45    4.83326E-01    6     1s  Added 1 cut on SDP cone  
   15 :   5.35460E-01     3.45    4.83371E-01    7     1s  Added 1 cut on SDP cone  
   16 :   4.96083E-01     0.85    4.83371E-01    8     1s  Improved solution found by heuristics | Added 1 cut on SDP cone  
* Finished.  Cost: 0.49608 (lower bound: 0.48337, relative gap 0.85337%)
* Termination with relative gap satisfied 
* Timing: 45% spent in upper solver (17 problems solved)
*         7% spent in lower solver (26 problems solved)
*         32% spent in LP-based domain reduction (174 problems solved)
*         3% spent in upper heuristics (175 candidates tried)
````

Beware, the BMI solver is absolutely not this efficient in general, this was just a lucky case. 

Let us solve a decay-rate maximization problem, also this a BMI problem. No feasible solution until after 12 nodes, and no lower bound available until the upper bound can be exploited to strenthen bounds on some of the unbounded variables (indicating that one should add artificial bounds, but we will improve the model instead).

````matlab
yalmip('clear');
A = [-1 2;-3 -4];
t = sdpvar(1,1);
P = sdpvar(2,2);
F = [P>=eye(2), A'*P+P*A <= -2*t*P];
F = [F, t >= 0];
options = sdpsettings('solver','bmibnb');
optimize(F,-t,options);
* Starting YALMIP global branch & bound.
* Upper solver     : fmincon
* Lower solver     : MOSEK
* LP solver        : GUROBI
* -Extracting bounds from model
* -Perfoming root-node bound propagation
* -Calling upper solver (no solution found)
* -Branch-variables : 4
* -More root-node bound-propagation
* -Performing LP-based bound-propagation 
* -And some more root-node bound-propagation
* Warning: 1 branch variable is unbounded from below
* Warning: 4 branch variables are unbounded from above
* Starting the b&b process
 Node       Upper       Gap(%)       Lower     Open   Time
    1 :           Inf      NaN           -Inf    2     0s  Added 1 cut on SDP cone  
    2 :           Inf      NaN           -Inf    3     0s  Added 1 cut on SDP cone  
    3 :           Inf      NaN           -Inf    4     0s  Added 1 cut on SDP cone  
    4 :           Inf      NaN           -Inf    5     0s  Added 1 cut on SDP cone  
    5 :           Inf      NaN           -Inf    6     0s    
    6 :           Inf      NaN           -Inf    7     0s    
    7 :           Inf      NaN           -Inf    8     0s    
    8 :           Inf      NaN           -Inf    7     0s  Infeasible in node bound-propagation  
    9 :           Inf      NaN           -Inf    6     0s  Infeasible in node bound-propagation  
   10 :           Inf      NaN           -Inf    5     0s  Infeasible in node bound-propagation  
   11 :           Inf      NaN           -Inf    6     1s    
   12 :  -2.18779E+00    22.80   -3.00795E+00    4     1s  Solution found by upper solver | Pruned stack based on new upper bound  
   ...
   40 :  -2.49940E+00     0.24   -2.50795E+00    2     2s  Improved solution found by upper solver | Pruned stack based on new upper bound  
* Finished.  Cost: -2.4994 (lower bound: -2.5079, relative gap 0.24403%)
* Termination with relative gap satisfied 
* Timing: 66% spent in upper solver (24 problems solved)
*         3% spent in lower solver (31 problems solved)
*         22% spent in LP-based domain reduction (201 problems solved)
*         1% spent in upper heuristics (103 candidates tried)
````

For this particular problem, the reason is easy to find. The original BMI is homogeneous, and to guarantee a somewhat reasonable solution, we artificially added the constraint \\(P \succeq I\\) instead of \\(P \succ 0\\) which theory prescribes (impossible in reality as strict inequalities are practical impossibility). A better model is obtained if we instead fix the trace of \\(P\\) to de-homogenize the model. This will make the feasible set w.r.t \\(P\\) bounded, but the problems are equivalent. With this, we go from a challenging problem to a trivial problem.

````matlab
F = [P >= 0, trace(P) == 1, A'*P+P*A <= -2*t*P];
F = [F,  t >= 0];
optimize(F,-t,options);
* Starting YALMIP global branch & bound.
* Upper solver     : fmincon
* Lower solver     : MOSEK
* LP solver        : GUROBI
* -Extracting bounds from model
* -Perfoming root-node bound propagation
* -Calling upper solver (no solution found)
* -Branch-variables : 4
* -More root-node bound-propagation
* -Performing LP-based bound-propagation 
* -And some more root-node bound-propagation
* Warning: 1 branch variable is unbounded from above
* Starting the b&b process
 Node       Upper       Gap(%)       Lower     Open   Time
    1 :  -2.50000E+00      NaN           -Inf    1     0s  Solution found by upper solver  
    2 :  -2.50000E+00     0.00   -2.50002E+00    2     0s  Improved solution found by upper solver  
* Finished.  Cost: -2.5 (lower bound: -2.5, relative gap 0.00045039%)
* Termination with relative gap satisfied 
* Timing: 51% spent in upper solver (3 problems solved)
*         14% spent in lower solver (10 problems solved)
*         20% spent in LP-based domain reduction (24 problems solved)
*         2% spent in upper heuristics (17 candidates tried)
````

For this problem, we can easily find more interesting cuts. The decay-rate BMI together with the constant trace implies \\(trace(A^TP+PA) \leq -2t\\). Adding this redundant cut intereestingly leads to a finite lower bound already in the root node, but branch & bound algorithms can be unpredictable in the path they take through the tree, and here it opens another node and ends up solving one extra node.

````matlab
F = [P>=0, trace(P)==1, A'*P+P*A <= -2*t*P];
F = [F,  t >= 0];
F = [F, trace(A'*P+P*A)<=-2*t]
optimize(F,-t,options);
* Starting YALMIP global branch & bound.
* Upper solver     : fmincon
* Lower solver     : MOSEK
* LP solver        : GUROBI
* -Extracting bounds from model
* -Perfoming root-node bound propagation
* -Calling upper solver (no solution found)
* -Branch-variables : 4
* -More root-node bound-propagation
* -Performing LP-based bound-propagation 
* -And some more root-node bound-propagation
* Starting the b&b process
    1 :  -2.50000E+00    16.93   -3.14753E+00    2     0s  Solution found by upper solver  
    2 :  -2.50000E+00    16.93   -3.14753E+00    1     0s  Terminated in bound propagation  
    3 :  -2.50000E+00     0.00   -2.50000E+00    0     0s  Numerical problems in lower solver | Added 1 cut on SDP cone  
* Finished.  Cost: -2.5 (lower bound: -2.5, relative gap 0%)
* Termination with all nodes pruned 
* Timing: 41% spent in upper solver (3 problems solved)
*         14% spent in lower solver (10 problems solved)
*         27% spent in LP-based domain reduction (25 problems solved)
*         3% spent in upper heuristics (22 candidates tried)
````

A Schur complement on the decay-rate BMI gives us yet another linear SDP cut which improves the root-node relaxation even more.

````matlab
F = [P>=0,A'*P+P*A <= -2*t*P, t >= 0];
F = [F, trace(P)==1];
F = [F, trace(A'*P+P*A)<=-2*t];
F = [F, [-A'*P-P*A P*t;P*t P*t/2] >= 0];
optimize(F,-t,options);
* Starting YALMIP global branch & bound.
* Upper solver     : fmincon
* Lower solver     : MOSEK
* LP solver        : GUROBI
* -Extracting bounds from model
* -Perfoming root-node bound propagation
* -Calling upper solver (no solution found)
* -Branch-variables : 4
* -More root-node bound-propagation
* -Performing LP-based bound-propagation 
* -And some more root-node bound-propagation
* Starting the b&b process
 Node       Upper       Gap(%)       Lower     Open   Time
    1 :  -2.50000E+00    10.91   -2.90400E+00    2     0s  Solution found by upper solver  
    2 :  -2.50000E+00    10.91   -2.90400E+00    1     0s  Terminated in bound propagation  
    3 :  -2.50000E+00     0.00   -2.50000E+00    0     0s  Numerical problems in lower solver | Added 2 cuts on SDP cone  
* Finished.  Cost: -2.5 (lower bound: -2.5, relative gap 0%)
* Termination with all nodes pruned 
* Timing: 46% spent in upper solver (3 problems solved)
*         18% spent in lower solver (10 problems solved)
*         23% spent in LP-based domain reduction (25 problems solved)
*         2% spent in upper heuristics (18 candidates tried)
````

By adding valid cuts, the relaxations are possibly tighter, leading to better lower bounds. A problem however is that we add additional burden to the local nonlinear solver used for the upper bounds. The additional cuts are redundant for the local solver, and most likely detoriate the performance. To avoid this, cuts can be explicitly specified by using the command [cut](/command/cut)(/command/cut). Constraints defined using this command will only be used in the solution of relaxations, and omitted when the local solver is called. Does not seem to make any difference here though (note that the story might be slightly different in the case of semidefinite constraints, since the semidefinite constraints are used for generating cuts for the nonlinear solver to improve SDP feasibility. Hence, if there are more SDP constraints in the model, other cuts will be generated for the nonlinear solver, meaning redundant SDP constraints may influence the upper bound process!)

````matlab
F = [P >=0 ,A'*P+P*A <= -2*t*P, t >= 0];
F = [F, trace(P) == 1];
F = [F, trace(A'*P+P*A) <= -2*t];
F = [F, cut([-A'*P-P*A P*t;P*t P*t/2] >= 0)];
optimize(F,-t,options);
* Starting YALMIP global branch & bound.
* Upper solver     : fmincon
* Lower solver     : MOSEK
* LP solver        : GUROBI
* -Extracting bounds from model
* -Perfoming root-node bound propagation
* -Calling upper solver (no solution found)
* -Branch-variables : 4
* -More root-node bound-propagation
* -Performing LP-based bound-propagation 
* -And some more root-node bound-propagation
* Starting the b&b process
 Node       Upper       Gap(%)       Lower     Open   Time
    1 :  -2.50000E+00    10.91   -2.90400E+00    2     0s  Solution found by upper solver  
    2 :  -2.50000E+00    10.91   -2.90400E+00    1     0s  Terminated in bound propagation  
    3 :  -2.50000E+00     0.00   -2.50000E+00    0     0s  Numerical problems in lower solver | Added 1 cut on SDP cone  
* Finished.  Cost: -2.5 (lower bound: -2.5, relative gap 0%)
* Termination with all nodes pruned 
* Timing: 41% spent in upper solver (3 problems solved)
*         23% spent in lower solver (10 problems solved)
*         26% spent in LP-based domain reduction (25 problems solved)
*         2% spent in upper heuristics (18 candidates tried)
````

Upper bounds re by default generated by the nonlinear cutting plane strategy. You can alternatively set the upper bound solver to [PENBMI/PENLAB](/solver/penbmi) which solves nonlinear SDP problems natively. Unfortunately though, these solvers are typically not robust enough to be used in the branch & bound scenario in [BMINB](/solver/bmibnb)

Finally, we note that the particular quasi-convex decay-rate problem we have studied here really shouldn't be solved using a full-fledged global solver, but is much more efficiently addressed using a [bisection](/solver/bisection).
