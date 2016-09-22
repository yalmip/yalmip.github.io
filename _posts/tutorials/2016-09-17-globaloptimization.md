---
title: "Global optimization"
category: tutorial
author_profile: false
level: 4
tags: [Global optimization, Nonconvex quadratic programming, Nonlinear semidefinite programming, BMI]
excerpt: "The holy grail! 60% of the time it works every time."
layout: single
sidebar:
  nav: "tutorials"
---

Global solutions! Well, don't expect too much from global solvers. 

The focus here is on the built-in solver [BMIBNB](/solver/bmibnb). The code is fairly robust on small problems (solves 180 of the globlib problems in under 8 minutes total), and a couple of small real-world problems with bilinear matrix inequalities have been solved successfully. Alternative global solvers can be found [here](/tags/#global-solver)

The [BMIBNB](/solver/bmibnb) solver is based on a simple spatial branch-and-bound strategy, using McCormick's convex envelopes for bounding bilinear terms, and general convex envelope approximations for other nonlinear operators. LP-based bound tightening is (optionally) applied iteratively to improve variable bounds together with some additional techniques to, e.g., exploit complementary constraints, extraction of bounds from quadratic, exploiting inverse functions etc. See the [envelope approximation](/tutorial/envelopesinbmibnb)) for some of the details.

Relaxed problems are solved using either an [LP solver](/tags/#linear-programming-solver), [QP solver](/tags/#quadratic-programming-solver), or [SDP solver](/tags/#semidefinite-programming-solver) solver, depending on the problem, while upper bounds are found using a local nonlinear solver such as [FMINCON](/solver/fmincon),  [SNOPT](/solver/snopt) and [IPOPT](/solver/ipopt), or [PENBMI/PENLAB](/solver/penbmi) for nonlinear semidefinite problems.

### Nonconvex quadratic programming
The first example is a problem with a concave quadratic constraint (this is the example addressed in the moment relaxation section). Three different optimization problems are solved during the branching: Upper bounds using a local nonlinear solver `'bmibnb.uppersolver'`, lower bounds with `'bmibnb.lowersolver'` and bound tightening using a linear programming solver `'bmibnb.lpsolver'`.

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
* Branch-variables : 3
* Upper solver     : fmincon
* Lower solver     : GUROBI
* LP solver        : GUROBI
 Node       Upper      Gap(%)       Lower    Open
    1 :   -4.000E+00    33.33     -6.000E+00   2  Improved solution  
    2 :   -4.000E+00    33.33     -6.000E+00   3    
    3 :   -4.000E+00    15.57     -4.844E+00   4    
    4 :   -4.000E+00    15.57     -4.844E+00   3  Infeasible  
    5 :   -4.000E+00    10.76     -4.569E+00   2  Poor bound  
    6 :   -4.000E+00    10.76     -4.569E+00   3    
    7 :   -4.000E+00     0.02     -4.001E+00   2  Infeasible  
* Finished.  Cost: -4 Gap: 0.020194
* Timing: 45% spent in upper solver (4 problems solved)
*         1% spent in lower solver (11 problems solved)
*         18% spent in LP-based domain reduction (106 problems solved)
````

The second example is a slightly larger nonconvex quadratic programming problem. The problem is easily solved to a gap of less than 1%.

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
* Branch-variables : 6
* Upper solver     : fmincon
* Lower solver     : GUROBI
* LP solver        : GUROBI
 Node       Upper      Gap(%)       Lower    Open
    1 :   -3.100E+02     0.96     -3.130E+02   2  Improved solution  
* Finished.  Cost: -310 Gap: 0.96
* Timing: 42% spent in upper solver (1 problems solved)
*         1% spent in lower solver (13 problems solved)
*         9% spent in LP-based domain reduction (12 problems solved)
````

Quadratic equality constraints is a common reason for nonconvexity, and can also be dealt with using the global solver. This can be used for, e.g., boolean programming (this is a very inefficient way to solve this simple MIQP and is only shown here to illustrate nonconvex equality constraints).

````matlab
n = 10;
x = sdpvar(n,1);

Q = randn(n,n);Q = Q*Q'/norm(Q)^2;
c = randn(n,1);

objective = x'*Q*x+c'*x;

F = [x.*(x-1)==0, 0<=x<=1];

options = sdpsettings('solver','bmibnb');
optimize(F,objective,options);
* Starting YALMIP global branch & bound.
* Branch-variables : 10
* Upper solver     : fmincon
* Lower solver     : GUROBI
* LP solver        : GUROBI
 Node       Upper      Gap(%)       Lower    Open
    1 :   -1.796E+00     1.84     -1.848E+00   2  Improved solution  
    2 :   -1.796E+00     1.84     -1.848E+00   1  Poor bound  
    3 :   -1.796E+00     0.00     -1.740E+00   0  Poor bound  
* Finished.  Cost: -1.7962 Gap: 0
* Timing: 41% spent in upper solver (1 problems solved)
*         4% spent in lower solver (23 problems solved)
*         5% spent in LP-based domain reduction (0 problems solved)
````

### Nonconvex polynomial programming

Polynomial programs are transformed to to bilinear programs. As an example, the variable \\(x^3y^2\\) will be replaced with the the variable \\(w\\) and the constraints \\(w = uv\\), \\(u = zx\\), \\(z = x^2\\), \\(v == y^2\\). This is done automatically if the global solver is called with a higher order polynomial problem. With this transformation, standard bilinear envelopes can be used in the creation of the relaxations for computing lower bounds.

````matlab
sdpvar x y
F = [x^3+y^5 <= 5, y >= 0];
F = [F, -100 <= [x y] <= 100]; % Always bounded domain
options = sdpsettings('verbose',1,'solver','bmibnb');
optimize(F,-x,options)
* Starting YALMIP global branch & bound.
* Branch-variables : 2
* Upper solver     : fmincon
* Lower solver     : GUROBI
* LP solver        : GUROBI
 Node       Upper      Gap(%)       Lower    Open
    1 :   -1.710E+00     0.00     -1.710E+00   0  Poor bound  
* Finished.  Cost: -1.71 Gap: 0
* Timing: 60% spent in upper solver (1 problems solved)
*         1% spent in lower solver (5 problems solved)
*         5% spent in LP-based domain reduction (4 problems solved)
````

### General nonconvex programming

The solver supports global optimization over almost all operators supported on decision variables in YALMIP, as most operators are equipped with convex envelope operators, and if not, they are generated on-the-fly using sample approximations. Hence, nothing prevents us from, e.g., nonconvex global trigonometric optimization

````matlab
sdpvar x y 
p = sin(1+y*x)^2+cos(y*x);
optimize([-1 <= [x y] <= 1],p,sdpsettings('solver','bmibnb'));
* Starting YALMIP global branch & bound.
* Branch-variables : 3
* Upper solver     : fmincon
* Lower solver     : GUROBI
* LP solver        : GUROBI
 Node       Upper      Gap(%)       Lower    Open
    1 :    5.403E-01     0.00      5.403E-01   2  Improved solution  
* Finished.  Cost: 0.5403 Gap: 3.7528e-06
* Timing: 53% spent in upper solver (1 problems solved)
*         1% spent in lower solver (7 problems solved)
*         8% spent in LP-based domain reduction (4 problems solved)
````



### Nonconvex semidefinite programming

The following problem is a classical BMI problem

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
* Branch-variables : 2
* Upper solver     : penlab
* Lower solver     : SeDuMi
* LP solver        : GUROBI
 Node       Upper      Gap(%)       Lower    Open
    1 :   -9.565E-01     2.20     -1.000E+00   2  Improved solution  
    2 :   -9.565E-01     2.20     -1.000E+00   3    
    3 :   -9.565E-01     0.00     -9.565E-01   0  Poor bound  
* Finished.  Cost: -0.95653 Gap: 0
* Timing: 58% spent in upper solver (2 problems solved)
*         9% spent in lower solver (7 problems solved)
*         5% spent in LP-based domain reduction (36 problems solved)
````

As a second BMI problem, we will solve a constrained LQR problem. For the global code to work, global lower and upper and bound on all complicating variables (involved in nonlinear terms) must be supplied, either explicitly or implicitly in the linear constraints. This was the case for all problems above. In this example, the variable **K** is already bounded in the original problem, but the elements in **P** have to be bounded artificially.

````matlab
yalmip('clear')
A = [-1 2;-3 -4];B = [1;1];
P = sdpvar(2,2);
K = sdpvar(1,2);
F = [P>=0, (A+B*K)'*P+P*(A+B*K) <= -eye(2)-K'*K];
F = [F, K<=0.1, K>=-0.1];
F = [F, 1000>=P(:)>=-1000];
options = sdpsettings('solver','bmibnb');
optimize(F,trace(P),options);
* Starting YALMIP global branch & bound.
* Branch-variables : 5
* Upper solver     : penlab
* Lower solver     : SeDuMi
* LP solver        : GUROBI
 Node       Upper      Gap(%)       Lower    Open
    1 :    4.834E-01    19.23      2.232E-01   2  Improved solution  
    2 :    4.834E-01    19.23      2.232E-01   1  Infeasible  
    3 :    4.834E-01     1.80      4.570E-01   2    
    4 :    4.834E-01     1.80      4.570E-01   1  Infeasible  
    5 :    4.834E-01     0.08      4.823E-01   2    
* Finished.  Cost: 0.48341 Gap: 0.076641
* Timing: 78% spent in upper solver (3 problems solved)
*         6% spent in lower solver (15 problems solved)
*         5% spent in LP-based domain reduction (72 problems solved)
````

Beware, the BMI solver is absolutely not that efficient in general, this was just a lucky case. Here is the decay-rate example instead (with some additional constraints on the elements in **P** to bound the feasible set).

````matlab
yalmip('clear');
A = [-1 2;-3 -4];
t = sdpvar(1,1);
P = sdpvar(2,2);
F = [P>=eye(2), A'*P+P*A <= -2*t*P];
F = [F, -1e4 <= P(:) <= 1e4, 100 >= t >= 0];
options = sdpsettings('solver','bmibnb');
optimize(F,-t,options);
* Starting YALMIP global branch & bound.
* Branch-variables : 4
* Upper solver     : penlab
* Lower solver     : SeDuMi
* LP solver        : GUROBI
 Node       Upper      Gap(%)       Lower    Open
    1 :          Inf      NaN     -9.984E+01   2    
    2 :          Inf      NaN     -9.984E+01   1  Infeasible  
    3 :   -2.500E+00   182.35     -7.483E+01   2  Improved solution  
    4 :   -2.500E+00   182.35     -7.483E+01   1  Infeasible  
 ...
   99 :   -2.500E+00    47.10     -4.656E+00   8  Infeasible  
  100 :   -2.500E+00    47.10     -4.656E+00   9    
* Finished.  Cost: -2.5 Gap: 47.101
* Timing: 94% spent in upper solver (54 problems solved)
*         4% spent in lower solver (106 problems solved)
*         2% spent in LP-based domain reduction (886 problems solved)
````

The linear relaxations give very poor lower bounds on this problem (but the optimal solution is found already in the third node), leading to poor convergence of the branching process. However, for this particular problem, the reason is easy to find. The original BMI is homogeneous w.r.t **P**, and to guarantee a somewhat reasonable solution, we artificially added the constraint **P>=I**. A better model is obtained if we instead fix the trace of **P**. This will make the feasible set w.r.t **P** much smaller, but the problems are equivalent. Note also that we no longer need any artificial constraints on the elements in **P**.

````matlab
F = [P>=0, A'*P+P*A <= -2*t*P, 100 >= t >= 0];
F = [F, trace(P)==1];
optimize(F,-t,options);
* Starting YALMIP global branch & bound.
* Branch-variables : 4
* Upper solver     : penlab
* Lower solver     : SeDuMi
* LP solver        : GUROBI
 Node       Upper      Gap(%)       Lower    Open
    1 :          Inf      NaN     -5.138E+01   2    
    2 :          Inf      NaN     -5.138E+01   1  Infeasible  
    3 :          Inf      NaN     -3.351E+00   2    
    4 :          Inf      NaN     -3.351E+00   3    
    5 :          Inf      NaN     -2.835E+00   4    
    6 :          Inf      NaN     -2.835E+00   3  Infeasible  
    7 :          Inf      NaN     -2.578E+00   2  Infeasible  
    8 :          Inf      NaN     -2.578E+00   3    
    9 :   -1.940E+00    19.51     -2.576E+00   4  Improved solution  
   10 :   -1.940E+00    19.51     -2.576E+00   3    
   11 :   -2.424E+00     4.06     -2.566E+00   4  Improved solution  
   12 :   -2.424E+00     4.06     -2.566E+00   1  Infeasible  
   13 :   -2.424E+00     2.66     -2.516E+00   2    
   14 :   -2.424E+00     2.66     -2.516E+00   3    
   15 :   -2.496E+00     0.14     -2.500E+00   4  Improved solution  
* Finished.  Cost: -2.4955 Gap: 0.13619
* Timing: 90% spent in upper solver (11 problems solved)
*         3% spent in lower solver (19 problems solved)
*         3% spent in LP-based domain reduction (118 problems solved)
````

For this problem, we can easily find a very efficient additional cutting plane. The decay-rate BMI together with the constant trace on **P** implies **trace(A'*P+P*A)<=-2t**. Adding this cut reduce the number of iterations needed.

````matlab
F = [P>=0,A'*P+P*A <= -2*t*P, 100 >= t >= 0];
F = [F, trace(P)==1];
F = [F, trace(A'*P+P*A)<=-2*t];
* Starting YALMIP global branch & bound.
* Branch-variables : 4
* Upper solver     : penlab
* Lower solver     : SeDuMi
* LP solver        : GUROBI
 Node       Upper      Gap(%)       Lower    Open
    1 :          Inf      NaN     -3.431E+00   2    
    2 :          Inf      NaN     -3.431E+00   1  Infeasible  
    3 :          Inf      NaN     -2.663E+00   2    
    4 :          Inf      NaN     -2.663E+00   3    
    5 :   -2.015E+00    18.34     -2.624E+00   4  Improved solution  
    6 :   -2.015E+00    18.34     -2.624E+00   3    
    7 :   -2.488E+00     0.48     -2.505E+00   4  Improved solution  
* Finished.  Cost: -2.4884 Gap: 0.47869
* Timing: 87% spent in upper solver (6 problems solved)
*         4% spent in lower solver (14 problems solved)
*         4% spent in LP-based domain reduction (71 problems solved)
````

A Schur complement on the decay-rate BMI gives us yet another cut which improves the node relaxation even more.

````matlab
F = [P>=0,A'*P+P*A <= -2*t*P,100 >= t >= 0];
F = [F, trace(P)==1];
F = [F, trace(A'*P+P*A)<=-2*t];
F = [F, [-A'*P-P*A P*t;P*t P*t/2] >= 0];
optimize(F,-t,options);
* Starting YALMIP global branch & bound.
* Branch-variables : 4
* Upper solver     : penlab
* Lower solver     : SeDuMi
* LP solver        : GUROBI
 Node       Upper      Gap(%)       Lower    Open
    1 :          Inf      NaN     -3.124E+00   2    
    2 :          Inf      NaN     -3.124E+00   3    
    3 :   -2.467E+00     1.66     -2.525E+00   4  Improved solution  
    4 :   -2.467E+00     1.66     -2.525E+00   1  Infeasible  
    5 :   -2.467E+00     1.06     -2.503E+00   2    
    6 :   -2.467E+00     1.06     -2.503E+00   1  Infeasible  
    7 :   -2.467E+00     1.00     -2.501E+00   2    
* Finished.  Cost: -2.4667 Gap: 0.9967
* Timing: 93% spent in upper solver (5 problems solved)
*         2% spent in lower solver (14 problems solved)
*         2% spent in LP-based domain reduction (83 problems solved)
````

By adding valid cuts, the relaxations are possibly tighter, leading to better lower bounds. A problem however is that we add additional burden to the local solver used for the upper bounds. The additional cuts are redundant for the local solver, and most likely detoriate the performance. To avoid this, cuts can be explicitly specified by using the command [cut](/command/cut). Constraints defined using this command will only be used in the solution of relaxations, and omitted when the local solver is called. Does not seem to make any difference here though.

````matlab
F = [P>=0,A'*P+P*A <= -2*t*P,100 >= t >= 0];
F = [F, trace(P)==1];
F = [F, trace(A'*P+P*A)<=-2*t];
F = [F, cut([-A'*P-P*A P*t;P*t P*t/2]>=0)];
optimize(F,-t,options);
* Starting YALMIP global branch & bound.
* Branch-variables : 4
* Upper solver     : penlab
* Lower solver     : SeDuMi
* LP solver        : GUROBI
 Node       Upper      Gap(%)       Lower    Open
    1 :          Inf      NaN     -3.124E+00   2    
    2 :          Inf      NaN     -3.124E+00   3    
    3 :   -2.467E+00     1.66     -2.525E+00   4  Improved solution  
    4 :   -2.467E+00     1.66     -2.525E+00   1  Infeasible  
    5 :   -2.467E+00     1.06     -2.503E+00   2    
    6 :   -2.467E+00     1.06     -2.503E+00   1  Infeasible  
    7 :   -2.467E+00     1.00     -2.501E+00   2    
* Finished.  Cost: -2.4667 Gap: 0.9967
* Timing: 87% spent in upper solver (5 problems solved)
*         4% spent in lower solver (14 problems solved)
*         3% spent in LP-based domain reduction (83 problems solved)
````

Upper bounds were obtained above by solving the BMI locally using [PENBMI/PENLAB](/solver/penbmi). If no local BMI solver is available, an alternative is to check if the relaxed solution is a feasible solution. If so, the upper bound can be updated. This scheme can be obtained by specifying 'none' as the upper bound solver. A solution within default tolerances is easily found.

````matlab
options = sdpsettings(options,'bmibnb.uppersolver','none');
F = [P>=0,A'*P+P*A <= -2*t*P,100 >= t >= 0];
F = [F, trace(P)==1];
F = [F, trace(A'*P+P*A)<=-2*t];
F = [F, cut([-A'*P-P*A P*t;P*t P*t/2]>=0)];
optimize(F,-t,options);
* Starting YALMIP global branch & bound.
* Branch-variables : 4
* Upper solver     : none
* Lower solver     : SeDuMi
* LP solver        : GUROBI
 Node       Upper      Gap(%)       Lower    Open
    1 :          Inf      NaN     -3.124E+00   2    
    2 :          Inf      NaN     -3.124E+00   3    
    3 :   -2.467E+00     1.66     -2.525E+00   4  Improved solution  
    4 :   -2.467E+00     1.66     -2.525E+00   1  Infeasible  
    5 :   -2.467E+00     1.06     -2.503E+00   2    
    6 :   -2.467E+00     1.06     -2.503E+00   1  Infeasible  
    7 :   -2.467E+00     1.00     -2.501E+00   2    
* Finished.  Cost: -2.4667 Gap: 0.9967
* Timing: 0% spent in upper solver (0 problems solved)
*         27% spent in lower solver (14 problems solved)
*         20% spent in LP-based domain reduction (83 problems solved)
````

Finally, we note that the particular quasi-convex problem we have studied here really shouldn't be solved using a full-fledged global solver, but is much more efficiently addressed using a [bisection](/solver/bisection)
