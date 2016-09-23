---
layout: single
category: example
author_profile: false
excerpt: "Optimization over optimization problems in three different ways."
title: Bilevel programming alternatives
tags: [Bilevel programming, Multi-parametric programming, kkt, solvebilevel]
comments: true
date: '2016-09-17'
sidebar:
  nav: "examples"
header:
  teaser: "bilevel.png"
---

YALMIP supports [bilevel programming natively](/tutorial/bilevelprogramming), but this example shows how simple bilevel problems can be solved by using other standard modules in YALMIP. We will illustrate three different ways to solve bilevel quadratic optimization problems exactly; a [multi-parametric programming](/tutorial/multiparametricprogramming) approach (which boils down to a mixed integer quadratic programming approach), a direct mixed-integer quadratic programming approach, and a global nonlinear programming approach.

The first part of this example requires linear and quadratic programming solvers, the second part a general nonlinear solver such as [FMINCON](/solver/fmincon), [SNOPT](/solver/snopt) or [IPOPT](/solver/ipopt), and the third part requires [MPT](/solver/mpt).

For an introduction to bilevel optimization, see [Bard 1999](/reference/bard1999).

### Bilevel quadratic programming

Our outer problem is a quadratic programming problem in the variables \\(x\\) and \\(z\\)

$$
\begin{aligned}
\text{minimize} ~~& \frac{1}{2}x^TQx+c^Tx + d^Tz \\
\text{subject to} ~~& Ax\leq b+Ez
\end{aligned}
$$

The variable \\(z\\) is constrained to be the optimal solution of an inner quadratic programming problem

$$
\begin{aligned}
z = \arg \min ~&\frac{1}{2}z^THz+e^Tz + f^Tx \\
\text{subject to}~& Fz\leq h+Gx
\end{aligned}
$$

The three approaches we will use all rely on the KKT conditions for the inner problem, but address this condition in different ways.

$$
\begin{aligned}
Hz+e+F^T\lambda &=0\\
\lambda &\geq 0\\
Fz&\leq h+Gx\\
\lambda^T(h+Gx-Fz) &=0
\end{aligned}
$$

Obviously, the reason bilevel quadratic optimization is hard, is the nonconvex complementary constraint that either the multiplier is zero, or the constraint is active.

We generate some data for a random instance .

````matlab
n = 3;
m = 2;

Q = randn(n,n);Q = Q*Q';
c = randn(n,1);
d = randn(m,1);
A = randn(15,n);
b = rand(15,1)*2*n;
E = randn(15,m);

H = randn(m,m);H = H*H';
e = randn(m,1);
f = randn(n,1);
F = randn(5,m);
h = rand(5,1)*2*m;
G = randn(5,n);
````

### Direct mixed integer quadratic programming approach

The complementary conditions simply say that the multiplier is zero, or the corresponding constraint is active. This can be modeled in YALMIP by using the logic programming module (or one can set this up manually using binary variables).

````matlab
x = sdpvar(n,1);
z = sdpvar(m,1);
lambda = sdpvar(length(h),1);
slack = h + G*x - F*z;

KKT = [H*z + e + F'*lambda == 0,
                       F*z <=  h + G*x,
                    lambda >= 0];

for i = 1:length(h)
 KKT = [KKT, ((lambda(i)==0) | (slack(i) == 0))];
end  
````

To derive a mixed integer programming model of the logic condition, bounds on all variables involved in the logic condition are necessary (this is the main problem with this formulation, it requires bounds on the multipliers)

````matlab
KKT = [KKT, lambda <= 100, -100 <= [x;z] <= 100];
````

Collect all constraints, and solve the outer problem (Note that the problem can be infeasible. If so, simply generate a new problem instance)

````matlab
optimize([KKT, A*x <= b + E*z], 0.5*x'*Q*x + c'*x + d'*z);
value(x)
value(z)
value(lambda)
value(slack)
````

Note that YALMIP has a built-in command for generating [kkt](/command/kkt), hence, the manually implemented model above can be simplified significantly.

### Nonlinear solver for complementary constraints

A more direct approach to handle the complementary constraints is to simply solve the nonlinear nonconvex problem that arise. To do this in YALMIP, we use the built-in global solver [BMIBNB](/solver/bmibnb). As before, the only addition compared to the theoretical KKT system is that we have to add explicit constraints in order to bound the search-space.

````matlab
KKT = [H*z + e + F'*lambda == 0,
                       F*z <=  h + G*x,
                    lambda >= 0,
       lambda.*(h+G*x-F*z) == 0];
KKT = [KKT, lambda <= 100, -100 <= [x;z] <= 100];

ops = sdpsettings('solver','bmibnb');
optimize([KKT, A*x <= b + E*z], 0.5*x'*Q*x + c'*x + d'*z,ops)
````

The performance of the global solver is fairly poor on this formulation. The reason is that it does not detect the complementary structure, but simply treats the problem as a general problem with bilinear constraints. To improve performance, we introduce a new variable for the slack and obtain an easily detected complementary structure. YALMIP will exploit this complementary structure to improve the bound propagation and branching process.

````matlab
slack = sdpvar(length(h),1);

KKT = [H*z + e + F'*lambda == 0,
                     slack ==  h + G*x - F*z,
                     slack >= 0,
                    lambda >= 0,                    
             lambda.*slack == 0];
KKT = [KKT, lambda <= 100, -100 <= [x;z] <= 100];

ops = sdpsettings('solver','bmibnb');
optimize([KKT, A*x <= b + E*z], 0.5*x'*Q*x + c'*x + d'*z,ops)
````

Alternatively, write the KKT using the built-in [complements]

````matlab
slack = sdpvar(length(h),1);

KKT = [H*z + e + F'*lambda == 0,
                     slack ==  h + G*x - F*z,
                     complements(slack >= 0,lambda >= 0),
KKT = [KKT, lambda <= 100, -100 <= [x;z] <= 100];

ops = sdpsettings('solver','bmibnb');
optimize([KKT, A*x <= b + E*z], 0.5*x'*Q*x + c'*x + d'*z,ops)
````


### Multiparametric programming approach

A more advanced way in YALMIP to solve this problem, is to explicitly compute a parametrized solution \\(z(x)\\) by using multiparametric programming. This will lead to a piecewise affine description of the optimizer, and when this expression is plugged into the outer problem, a mixed integer quadratic programming problem arise.

Hence, we solve the inner program multiparametrically w.r.t. \\(x\\). Notice that we add bounds on \\(x\\), to limit the region where we are interested in a parametric solution (this is required for the parametric solver in [MPT](/solver/mpt) to perform well.)

````matlab
obj_inner = 0.5*z'*H*z + e'*z + f'*x;
cst_inner = [F*z <= h + G*x, -100 <= x <= 100];
[aux1,aux2,aux3,OptVal,OptZ] = solvemp(cst_inner,obj_inner,[],x);
````

At this point, the variable **OptZ** defines a piecewise affine function corresponding to the optimizing **z**. We use this variable in our outer problem

````matlab
obj_outer = 0.5*x'*Q*x + c'*x + d'*OptZ;
cst_outer = [A*x <= b + E*OptZ];
optimize(cst_outer,obj_outer);
value(x)
value(OptZ)
````

The multiparametric solver will essentially explore the combinatorial combinations of the active sets, and return a piecewise affine optimizer in each optimal combination. We can do this exploration manually, by simply stating the complementary conditions using logic constraints, and that is exactly what we did with the first approach above.

Note though, having the complete multiparametric solution, we might just as well loop through all regions and solve the problem with the corresponding inner optimal parametrization

````matlab
minn = inf;
for i = 1:length(aux1{1}.Pn)    
    OptZ = aux1{1}.Fi{i}*x + aux1{1}.Gi{i};
    obj_outer = 0.5*x'*Q*x + c'*x + d'*OptZ;
    cst_outer = [A*x <= b + E*OptZ, ismember(x,aux1{1}.Pn(i))];
    sol = optimize(cst_outer,obj_outer);
    if sol.problem == 0
        if value(obj_outer) < minn
            xsol2 = value(x);
            zsol2 = value(OptZ);
            minn = value(obj_outer);
        end
    end
end
````

### Using the built-in bilevel solver

As we mentioned above, YALMIP has a [built-in bilevel solver](/solver/solvebilevel) which applies to this problem.

````matlab
con_inner = F*z <=  h + G*x;
obj_inner = 0.5*z'*H*z+e'*z;
con_outer = A*x <= b + E*z;
obj_outer = 0.5*x'*Q*x + c'*x + d'*z;
solvebilevel(con_outer,obj_outer,con_inner,obj_inner,z)
````

By default, it solves the problem by explicitly branching on the complementarity, but we can tell it to define the [kkt](/command/kkt) model and solve the problem using integer solver, as we did above

````matlab
solvebilevel(con_outer,obj_outer,con_inner,obj_inner,z,sdpsettings('bilevel.algorithm','external'))
````
