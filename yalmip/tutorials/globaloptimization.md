---
title: "Global optimization"
layout: single
sidebar:
  nav: "tutorials"
---

Global solutions! Well, almost... don't expect too much at this stage. The solver used here, [bmibnb](/yalmip/solvers/bmibnb), is under development. The code is fairly robust on small problems (solves 180 of the globlib problems in under 8 minutes total), and a couple of small real-world problems with bilinear matrix inequalities have been solved successfully.

The [BMIBNBTheory] is based on a simple spatial branch-and-bound strategy, using McCormick's convex envelopes for bounding bilinear terms, and general convex envelope approximations for other nonlinear operators. LP-based bound tightening is applied iteratively to improve variable bounds together with some additional techniques to, e.g., exploit complementary constraints etc. See the [[BMIBNBTheory] for some details.

Relaxed problems are solved using either an [LP solver], [QP solver], or an [SDP solver] solver, depending on the problem, while upper bounds are found using a local nonlinear solver such as [FMINCON](/yalmip/solvers/fmincon),  [SNOPT](/yalmip/solvers/snopt) and [IPOPT](/yalmip/solvers/ipopt), or [PENBMI/PENLAB](/yalmip/solvers/penbmi) for nonlinear semidefinite problems.

### Nonconvex quadratic programming
The first example is a problem with a concave quadratic constraint (this is the example addressed in the moment relaxation section). Three different optimization problems are solved during the branching: Upper bounds using a local nonlinear solver `'bmibnb.uppersolver'`, lower bounds with `'bmibnb.lowersolver'` and bound tightening using a linear programming solver (`'bmibnb.lpsolver'`).

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
* Starting YALMIP bilinear branch & bound.
* Lower solver   : glpk
* Upper solver   : penbmi
 Node       Upper      Gap(%)       Lower    Open
    1 :          Inf      NaN    -6.000E+000   2    
    2 :          Inf      NaN    -6.000E+000   3    
    3 :  -4.000E+000    40.00    -6.000E+000   4  Improved solution  
    4 :  -4.000E+000    40.00    -6.000E+000   3  Infeasible  
    5 :  -4.000E+000    37.98    -5.899E+000   4  Poor bound  
    6 :  -4.000E+000    37.98    -5.899E+000   5    
    7 :  -4.000E+000    25.80    -5.290E+000   6    
    8 :  -4.000E+000    25.80    -5.290E+000   5  Infeasible  
    9 :  -4.000E+000     7.08    -4.354E+000   4  Infeasible  
   10 :  -4.000E+000     7.08    -4.354E+000   3  Infeasible  
   11 :  -4.000E+000     0.06    -4.003E+000   4    
+  11 Finishing.  Cost: -4 Gap: 0.06486%
````

The second example is a slightly larger problem indefinite quadratic programming problem. The problem is easily solved to a gap of less than 1%.

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
* Starting YALMIP bilinear branch & bound.
* Lower solver   : glpk
* Upper solver   : penbmi
 Node       Upper      Gap(%)       Lower    Open
    1 :          Inf      NaN    -3.130E+002   2    
    2 :  -3.100E+002     0.96    -3.130E+002   3  Improved solution  
+   2 Finishing.  Cost: -310 Gap: 0.96463%
````

Quadratic equality constraints is a common reason for nonconvexity, but can also be dealt with using the global solver. This can be used for, e.g., boolean programming (this is a very inefficient way to solve this simple MIQP and is only shown here to illustrate nonconvex equality constraints).

````matlab
n = 10;
x = sdpvar(n,1);

Q = randn(n,n);Q = Q*Q'/norm(Q)^2;
c = randn(n,1);

objective = x'*Q*x+c'*x;

F = [x.*(x-1)==0, 0<=x<=1];

options = sdpsettings('solver',bmibnb');
optimize(F,objective,options);
* Starting YALMIP bilinear branch & bound.
* Lower solver   : glpk
* Upper solver   : penbmi
 Node       Upper      Gap(%)       Lower    Open
    1 :  -2.967E+000     0.00    -2.967E+000   2  Improved solution  
+   1 Finishing.  Cost: -2.9671 Gap: 2.5207e-009%
````

### Nonconvex polynomial programming

Polynomial programs are transformed to to bilinear programs. As an example, the variable \\(x^3y^2\\) will be replaced with the the variable \\(w\\) and the constraints \\(w = uv\\), \\(u = zx\\), \\(z = x^2\\), \\(v == y^2\\). This is done automatically if the global solver is called with a higher order polynomial problem. With this transformation, standard bilinear envelopes can be used in the creation of the relaxations for computing lower bounds.

````matlab
sdpvar x y
F = [x^3+y^5 <= 5, y >= 0];
F = [F, -100 <= [x y] <= 100]; % Always bounded domain
options = sdpsettings('verbose',1,'solver','bmibnb');
optimize(F,-x,options)
* Starting YALMIP bilinear branch & bound.
* Lower solver   : glpk
* Upper solver   : penbmi
 Node       Upper      Gap(%)       Lower    Open
    1 :          Inf      NaN    -2.990E+001   2    
    2 :          Inf      NaN    -2.990E+001   1  Infeasible  
    3 :          Inf      NaN    -2.763E+001   2    
    4 :          Inf      NaN    -2.763E+001   3    
    5 :          Inf      NaN    -1.452E+001   4    
    6 :          Inf      NaN    -1.452E+001   5    
    7 :  -1.710E+000   171.54    -6.359E+000   4  Improved solution  
    8 :  -1.710E+000   171.54    -6.359E+000   3  Infeasible  
    9 :  -1.710E+000   127.11    -5.155E+000   4  Poor bound  
   10 :  -1.710E+000   127.11    -5.155E+000   3  Infeasible  
   11 :  -1.710E+000     0.00    -1.710E+000   2  Infeasible  
+  11 Finishing.  Cost: -1.71 Gap: 1.247e-005%
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
options = sdpsettings('bmibnb.lowersolver','pensdp','bmibnb.uppersolver','penlab');
options = sdpsettings(options,'solver','bmibnb','bmibnb.lpsolver','glpk');
options = sdpsettings(options,'verbose',2,'solver','bmibnb');
optimize(F,t,options);
* Starting YALMIP bilinear branch & bound.
* Lower solver   : pensdp
* Upper solver   : penbmi
 Node       Upper      Gap(%)       Lower    Open
    1 :  -9.565E-001     2.22    -1.000E+000   2  Improved solution  
    2 :  -9.565E-001     2.22    -1.000E+000   1  Infeasible  
    3 :  -9.565E-001     2.22    -1.000E+000   2    
    4 :  -9.565E-001     2.22    -1.000E+000   3    
    5 :  -9.565E-001     0.24    -9.613E-001   2  Infeasible  
+   5 Finishing.  Cost: -0.95653 Gap: 0.24126%
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
options = sdpsettings('verbose',1,'solver','bmibnb');
optimize(F,trace(P),options);
* Starting YALMIP bilinear branch & bound.
* Lower solver   : pensdp
* Upper solver   : penbmi
 Node       Upper      Gap(%)       Lower    Open
    1 :   4.834E-001    17.70     2.209E-001   2  Improved solution  
    2 :   4.834E-001    17.70     2.209E-001   1  Infeasible  
    3 :   4.834E-001     1.93     4.548E-001   2    
    4 :   4.834E-001     1.93     4.548E-001   1  Infeasible  
    5 :   4.834E-001     0.25     4.797E-001   2    
+   5 Finishing.  Cost: 0.48341 Gap: 0.25032%
````

Beware, the BMI solver is absolutely not that efficient in general, this was just a lucky case. Here is the decay-rate example instead (with some additional constraints on the elements in **P** to bound the feasible set).

````matlab
yalmip('clear');
A = [-1 2;-3 -4];
t = sdpvar(1,1);
P = sdpvar(2,2);
F = [P>eye(2), A'*P+P*A <= -2*t*P];
F = [F, -1e4 <= P(:) <= 1e4, 100 >= t >= 0];
options = sdpsettings('verbose',1,'solver','bmibnb');
optimize(F,-t,options);
* Starting YALMIP bilinear branch & bound.
* Lower solver   : pensdp
* Upper solver   : penbmi
 Node       Upper      Gap(%)       Lower    Open
    1 :          Inf      NaN    -9.965E+001   2    
    2 :  -2.500E+000  2775.83    -9.965E+001   3  Improved solution  
    3 :  -2.500E+000  2721.00    -9.773E+001   4    
    4 :  -2.500E+000  2721.00    -9.773E+001   3  Infeasible  
    5 :  -2.500E+000  2634.59    -9.471E+001   4    
    6 :  -2.500E+000  2634.59    -9.471E+001   3  Infeasible  
    7 :  -2.500E+000  2577.61    -9.272E+001   4    
    8 :  -2.500E+000  2577.61    -9.272E+001   3  Infeasible  
    9 :  -2.500E+000  2501.03    -9.004E+001   4    
   10 :  -2.500E+000  2501.03    -9.004E+001   3  Infeasible  
 ...
   94 :  -2.500E+000     4.30    -2.650E+000  49    
   95 :  -2.500E+000     0.47    -2.516E+000  50    
+  95 Finishing.  Cost: -2.5 Gap: 0.46843%
````

The linear relaxations give very poor lower bounds on this problem, leading to poor convergence of the branching process. However, for this particular problem, the reason is easy to find. The original BMI is homogeneous w.r.t **P**, and to guarantee a somewhat reasonable solution, we artificially added the constraint **P>=I**. A better model is obtained if we instead fix the trace of **P**. This will make the feasible set w.r.t **P** much smaller, but the problems are equivalent. Note also that we no longer need any artificial constraints on the elements in **P**.

````matlab
F = [P>=0, A'*P+P*A <= -2*t*P, 100 >= t >= 0];
F = [F, trace(P)==1];
optimize(F,-t,options);
* Starting YALMIP bilinear branch & bound.
* Lower solver   : pensdp
* Upper solver   : penbmi
 Node       Upper      Gap(%)       Lower    Open
    1 :  -2.500E+000  1396.51    -5.138E+001   2  Improved solution  
    2 :  -2.500E+000  1396.51    -5.138E+001   1  Infeasible  
    3 :  -2.500E+000     1.41    -2.549E+000   2    
    4 :  -2.500E+000     1.41    -2.549E+000   1  Infeasible  
    5 :  -2.500E+000     0.17    -2.506E+000   2    
+   5 Finishing.  Cost: -2.5 Gap: 0.1746%
````

For this problem, we can easily find a very efficient additional cutting plane. The decay-rate BMI together with the constant trace on **P** implies **trace(A'*P+P*A)<=-2t**. Adding this cut reduce the number of iterations needed.

````matlab
F = [P>0,A'*P+P*A <= -2*t*P,100 >= t >= 0];
F = [F, trace(P)==1];
F = [F, trace(A'*P+P*A)<=-2*t];
optimize(F,-t,options);
* Starting YALMIP bilinear branch & bound.
* Lower solver   : pensdp
* Upper solver   : penbmi
 Node       Upper      Gap(%)       Lower    Open
    1 :  -2.500E+000    26.60    -3.431E+000   2  Improved solution  
    2 :  -2.500E+000    26.60    -3.431E+000   3    
    3 :  -2.500E+000     0.55    -2.519E+000   2  Infeasible  
+   3 Finishing.  Cost: -2.5 Gap: 0.5461%
````

A Schur complement on the decay-rate BMI gives us yet another cut which improves the node relaxation even more.

````matlab
F = [P>0,A'*P+P*A <= -2*t*P,100 >= t >= 0];
F = [F, trace(P)==1];
F = [F, trace(A'*P+P*A)<=-2*t];
F = [F, [-A'*P-P*A P*t;P*t P*t/2] >= 0];
optimize(F,-t,options);
* Starting YALMIP bilinear branch & bound.
* Lower solver   : pensdp
* Upper solver   : penbmi
 Node       Upper      Gap(%)       Lower    Open
    1 :  -2.500E+000    17.84    -3.124E+000   2  Improved solution  
    2 :  -2.500E+000    17.84    -3.124E+000   3    
    3 :  -2.500E+000     0.55    -2.519E+000   2  Infeasible  
+   3 Finishing.  Cost: -2.5 Gap: 0.55275%
````

By adding valid cuts, the relaxations are possibly tighter, leading to better lower bounds. A problem however is that we add additional burden to the local solver used for the upper bounds. The additional cuts are redundant for the local solver, and most likely detoriate the performance. To avoid this, cuts can be explicitly specified by using the command [cut](/yalmip/commands/cut). Constraints defined using this command will only be used in the solution of relaxations, and omitted when the local solver is called.

````matlab
F = [P>=0,A'*P+P*A <= -2*t*P,100 >= t >= 0];
F = [F, trace(P)==1];
F = [F, trace(A'*P+P*A)<=-2*t];
F = [F, cut([-A'*P-P*A P*t;P*t P*t/2]>=0)];
optimize(F,-t,options);
* Starting YALMIP bilinear branch & bound.
* Lower solver   : pensdp
* Upper solver   : penbmi
 Node       Upper      Gap(%)       Lower    Open
    1 :  -2.500E+000    17.84    -3.124E+000   2  Improved solution  
    2 :  -2.500E+000    17.84    -3.124E+000   3    
    3 :  -2.500E+000     0.55    -2.519E+000   2  Infeasible  
+   3 Finishing.  Cost: -2.5 Gap: 0.55275%
````

Upper bounds were obtained above by solving the BMI locally using [PENBMI/PENLAB](/yalmip/solvers/penbmi). If no local BMI solver is available, an alternative is to check if the relaxed solution is a feasible solution. If so, the upper bound can be updated. This scheme can be obtained by specifying 'none' as the upper bound solver.

````matlab
options = sdpsettings(options,'bmibnb.uppersolver','none');
F = [P>=0,A'*P+P*A <= -2*t*P,100 >= t >= 0];
F = [F, trace(P)==1];
F = [F, trace(A'*P+P*A)<=-2*t];
F = [F, cut([-A'*P-P*A P*t;P*t P*t/2]>=0)];
optimize(F,-t,options);
* Starting YALMIP bilinear branch & bound.
* Lower solver   : pensdp
* Upper solver   : none
 Node       Upper      Gap(%)       Lower    Open
    1 :          Inf      NaN    -3.124E+000   2    
    2 :          Inf      NaN    -3.124E+000   3    
    3 :  -9.045E-001   104.47    -2.894E+000   4  Improved solution  
    4 :  -9.045E-001   104.47    -2.894E+000   5    
    5 :  -1.467E+000    52.43    -2.761E+000   4  Improved solution  
    6 :  -1.467E+000    52.43    -2.761E+000   5    
    7 :  -1.831E+000    29.87    -2.677E+000   4  Improved solution  
    8 :  -1.831E+000    29.87    -2.677E+000   5    
    9 :  -2.071E+000    17.77    -2.616E+000   4  Improved solution  
   10 :  -2.071E+000    17.77    -2.616E+000   5    
   11 :  -2.229E+000    10.46    -2.567E+000   4  Improved solution  
   12 :  -2.229E+000    10.46    -2.567E+000   5    
   13 :  -2.332E+000     5.70    -2.522E+000   4  Improved solution  
   14 :  -2.332E+000     5.70    -2.522E+000   5    
   15 :  -2.397E+000     3.25    -2.508E+000   4  Improved solution  
   16 :  -2.397E+000     3.25    -2.508E+000   5    
   17 :  -2.435E+000     2.01    -2.504E+000   4  Improved solution  
   18 :  -2.435E+000     2.01    -2.504E+000   5    
   19 :  -2.459E+000     1.25    -2.503E+000   4  Improved solution  
   20 :  -2.459E+000     1.25    -2.503E+000   5    
   21 :  -2.478E+000     0.70    -2.502E+000   4  Improved solution  
+  21 Finishing.  Cost: -2.4776 Gap: 0.69631%
````
