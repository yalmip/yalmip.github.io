---
title: "Sum-of-squares programming"
category: tutorial
author_profile: false
level: 4
tags: [Sum-of-squares programming, Semidefinite programming, Polynomials]
excerpt: "Almost nothing is a sum-of-squares, but let's hope yours is."
layout: single
sidebar:
  nav: "tutorials"
---

> The robust optimization module is described in the paper (https://yalmip.github.io/reference/lofberg2009/)[Löfberg 2009] (which should be cited if you use this functionality).


YALMIP has a built-in module for sum-of-squares calculations. In its most basic formulation, a sum-of-squares (SOS) problem takes a polynomial \\(p(x)\\) and tries to find a real-valued polynomial vector function \\(h(x)\\) such that \\(p(x)=h^T(x)h(x)\\) (or equivalently \\(p(x)=v^T(x)Qv(x)\\) where \\(Q\\) is positive semidefinite and \\(v(x)\\) is a vector of monomials), hence proving non-negativity of the polynomial \\(p(x)\\). Read more about standard sum-of-squares decompositions in [Parrilo 2003].

In addition to standard SOS decompositions which we discuss below, YALMIP also supports linearly and nonlinearly parameterized problems, decomposition of matrix valued polynomials, [Examples.MoreSOS symmetry reduction], [Examples.MoreSOS pre- and post-processing],  and computation of low-rank decompositions. These extension are described in [Löfberg 2009].

### Doing it your self the hard way

Before we introduce the efficiently implemented SOS module in YALMIP, let us briefly mention how one could implement a SOS solver in high-level YALMIP code. Of course, you would never use this approach, but it might be useful to see the basic idea. Define a polynomial.

````matlab
x = sdpvar(1,1);y = sdpvar(1,1);
p = (1+x)^4 + (1-y)^2;
````

Introduce a decomposition \\(p = v^TQv\\)

````matlab
v = monolist([x y],degree(p)/2);
Q = sdpvar(length(v));
p_sos = v'*Q*v;
````

The polynomials have to match, hence all coefficient in the polynomial describing the difference of the two polynomials have to be zero.

````matlab
F = [coefficients(p-p_sos,[x y]) == 0, Q >= 0];
optimize(F)
````

The problem above is typically more efficiently solved when interpreted in primal form, hence we [dualize](/command/dualize) it.

````matlab
F = [coefficients(p-p_sos,[x y]) == 0, Q >= 0];
optimize(F,[],sdpsettings('dualize',1))
````

Adding parametrizations does not change the code significantly. Here is the code to find a lower bound on the polynomial

````matlab
sdpvar t
F = [coefficients((p-t)-p_sos,[x y]) == 0, Q >= 0];
optimize(F,-t)
````

Matrix valued decompositions are a bit trickier, but still straightforward.

````matlab
sdpvar x y
P = [1+x^2 -x+y+x^2;-x+y+x^2 2*x^2-2*x*y+y^2];
m = size(P,1);
v = monolist([x y],degree(P)/2);
Q = sdpvar(length(v)*m);
R = kron(eye(m),v)'*Q*kron(eye(m),v)-P;
s = coefficients(R(find(triu(R))),[x y]);
optimize([Q >= 0, s==0]);
sdisplay(clean(kron(eye(m),v)'*value(Q)*kron(eye(m),v),1e-6))
````

Once again, this is the basic idea behind the SOS module in YALMIP. However, the implementation is far more efficient and uses various approaches to reduce complexity, hence the approaches above should never be used in practice.

### Sum-of-squares optimization

The following lines of code presents some typical manipulations when working with SOS-calculations.

The most important commands are [sos](/command/sos) (to define a SOS constraint) and [solvesos](/command/solvesos) (to solve the problem)

````matlab
x = sdpvar(1,1);y = sdpvar(1,1);
p = (1+x)^4 + (1-y)^2;
F = sos(p);
solvesos(F);
````

The sum-of-squares decomposition is extracted with the command [sosd](/command/sosd).

````matlab
h = sosd(F);
sdisplay(h)

ans =
  '-1.203-1.9465x+0.22975y-0.97325x^2'
  '0.7435-0.45951x-0.97325y-0.22975x^2'
  '0.0010977+0.00036589x+0.0010977y-0.0018294x^2'
````

To see if the decomposition was successful, we simply calculate the error \\(p(x)-h^T(x)h(x)\\) which should be 0. However, due to the way SDPs are solved, the difference will typically not be zero. A useful command then is [clean](/command/clean). Using [clean](/command/clean), we remove all monomials with coefficients smaller than, e.g., 1e-6.

````matlab
clean(p-h'*h,1e-6)

ans =
    0
````

The decomposition \\(p(x)=v^T(x)Qv(x)\\) can also be obtained easily.

````matlab
x = sdpvar(1,1);y = sdpvar(1,1);
p = (1+x)^4 + (1-y)^2;
F = sos(p);
[sol,v,Q] = solvesos(F);
clean(p-v{1}'*Q{1}*v{1},1e-6)
 ans =
     0
````

Note: Even though \\(h^T(x)h(x)\\)  and \\(v^T(x)Qv(x)\\)  should be the same in theory, they are typically not. The reason is partly numerical issues with floating point numbers, but more importantly due to the way YALMIP treats the case when \\(Q\\) not is positive definite (sometimes the case due to numerical issues in the SDP solver). The decomposition is in theory typically defined as \\(h(x)=\text{chol}(Q)v(x)\\). YALMIP however uses a decomposition based on a singular value decomposition to avoid problems in the singular and numerically indefinite case. If \\(Q\\) is positive definite the two expressions coincide.

The quality of the decomposition can alternatively be evaluated using [check](/command/check). The value reported is the largest coefficient in the polynomial  \\(p(x)-v^T(x)Qv(x)\\)

````matlab
checkset(F)
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
|   ID|      Constraint|                          Type|   Primal residual|
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
|   #1|   Numeric value|   SOS constraint (polynomial)|       7.3674e-012|
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
e = checkset(F(is(F,'sos')))

e =
  7.3674e-012
````

Easiest way however is to use a fourth output in the call to [solvesos](/command/solvesos)

````matlab
[sol,v,Q,res] = solvesos(F);
res =
  7.3674e-012
````

It is very rare (except in contrived academic examples) that one finds a decomposition with a positive definite \\(Q\\) and zero mismatch between \\(v^T(x)Qv(x)\\)  and the polynomial \\(p(x)\\). The reason is simply that all SDP solver use floating-point arithmetics. Hence, in principle, the decomposition has no theoretical value as a certificate for non-negativity unless additional post-analysis is performed (relating the size of the residual with the eigenvalues of the Gramian).

### Parameterized problems

The involved polynomials can be parametrized, and we can optimize the parameters under the constraint that the polynomial is a sum-of-squares. As an example, we can calculate a lower bound on a polynomial. The second argument can be used for an objective function to be minimized. Here, we maximize the lower bound, i.e. minimize the negative lower bound. The third argument is an options structure while the fourth argument is a vector containing all parametric variables in the polynomial (in this example we only have one variable, but several parametric variables can be defined by simply concatenating them).

````matlab
sdpvar x y lower
p = (1+x*y)^2-x*y+(1-y)^2;
F = sos(p-lower);
solvesos(F,-lower,[],lower);
value(lower)

 ans =
     0.75
````

YALMIP automatically declares all variables appearing in the objective function and in non-SOS constraints as parametric variables. Hence, the following code is equivalent.

````matlab
solvesos(F,-lower);
value(lower)

 ans =
     0.75
````

Multiple SOS constraints can also be used. Consider the following problem of finding the smallest possible \\(t\\) such that the polynomials \\(t(1+xy)^2-xy+(1-y)^2\\) and \\((1-xy)^2+xy+t(1+y)^2\\) are both sum-of-squares.

````matlab
sdpvar x y t
p1 = t*(1+x*y)^2-x*y+(1-y)^2;
p2 = (1-x*y)^2+x*y+t*(1+y)^2;
F = [sos(p1), sos(p2)];
solvesos(F,t);
value(t)

 ans =

   0.2500
sdisplay(sosd(F(1)))
ans =
    '-1.102+0.95709y+0.14489xy'
    '-0.18876-0.28978y+0.47855xy'
sdisplay(sosd(F(2)))
ans =
    '-1.024-0.18051y+0.76622xy'
    '-0.43143-0.26178y-0.63824xy'
    '0.12382-0.38586y+0.074568xy'
````


If you have parametric variables, bounding the feasible region typically improves numerical behavior. Having lower bounds will additionally decrease the size of the optimization problem (variables bounded from below can be treated as translated cone variables in [dualization], which is used by [solvesos](/command/solvesos)).

One of the most common mistake people make when using the sum-of-squares module is to forget to declare some parametric variables. This will typically lead to a (of course erroneous) huge sum-of-squares problem which easily freezes MATLAB and/or give strange error.

### Constrained polynomial optimization

The sum-of-squares module in YALMIP only deals with the most basic problem; proving positivity of a polynomial over \\(\mathbf{R}^n\\). If you want to check positivity over a semi-algebraic set, you have to formulate the suitable sum-of-squares formulation. The trick to do that is sometimes variable transformations (such as defining $x = u^2$ when optimizing over non-negative numbers), but in most cases application of the positivstellensatz, which can be seen as a generalization of the S-procedure. Roughly speaking, to show that \\(p(x)\\) is positive on the set \\(g(x)\geq 0\\), we can search for a non-negative polynomial \\(s(x)\\) such that \\(p(x)\geq g(x)s(x)\\), a trivially sufficient condition. To get a tractable problem, we replace non-negativity with sum-of-squares conditions. Let us use this technique to solve the simple lower bound computation from above, but this time constrained to the unit-box. Define the variables from before, and the constraints defining the unit-box.

````matlab
sdpvar x y lower
p = (1+x*y)^2-x*y+(1-y)^2;
g = [1-x;1+x;1-y;1+y]
````

Define 4 parametrized polynomials. Let us hope quadratic multipliers suffice (if results are unsatisfactory, we can use higher order multipliers, leading to, of course, larger problems)

````matlab
[s1,c1] = polynomial([x y],2);
[s2,c2] = polynomial([x y],2);
[s3,c3] = polynomial([x y],2);
[s4,c4] = polynomial([x y],2);
````

Apply the positivstellensatz and try to find coefficients of the multiplier polynomials to maximize the lower bound

````matlab
F = [sos(p-lower-[s1 s2 s3 s4]*g), sos(s1), sos(s2), sos(s3), sos(s4)];
solvesos(F,-lower,[],[c1;c2;c3;c4;lower]);
````
