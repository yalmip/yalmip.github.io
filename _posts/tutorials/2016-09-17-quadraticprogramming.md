---
title: "Quadratic programming"
category: tutorial
author_profile: false
tags: [Quadratic programming, Large-scale quadratic programming]
excerpt: "Almost as easy as linear programming. Be careful though, symbolics might start to cause overhead."
layout: single
level: 2
sidebar:
  nav: "tutorials"
---

Let us assume that we have data generated from a noisy linear regression \\(y_t = a_tx + e_t\\). The goal is to estimate the parameter \\(x\\), given the measurements \\(y_t\\) and \\(a_t\\), and we will try 3 different approaches based on linear and quadratic programming.

Create some noisy data with severe outliers to work with.

````matlab
x = [1 2 3 4 5 6]';
t = (0:0.02:2*pi)';
A = [sin(t) sin(2*t) sin(3*t) sin(4*t) sin(5*t) sin(6*t)];
e = (-4+8*rand(length(t),1));
e(100:115) = 30;
y = A*x+e;
plot(t,y);
````

![Data for regression problem]({{ site.url }}/images/regressdata.png){: .center-image }

Define the variable we are looking for

````matlab
xhat = sdpvar(6,1);
````

By using **xhat** and the regressors in **A**, we can define the residuals (which also will be an [sdpvar](/yalmip/commands/sdpvar) object, parametrized in **xhat**)

````matlab
residuals = y-A*xhat;
````

To solve the 1-norm regression problem (minimize sum of absolute values of residuals), we can define a variable that will serve as a bound on the absolute values of **y-A*xhat** (we will solve this problem much more conveniently below by simply using the norm operator)

````matlab
bound = sdpvar(length(residuals),1);
````

Express that the bound variables are larger than the absolute values of the residuals (note the convenient definition of a double-sided constraint).

````matlab
Constraints = [-bound <= residuals <= bound];
````

Call YALMIP to minimize the sum of the bounds subject to the constraints in **Constraints**. YALMIP will automatically detect that this is a linear program, and call any [LP solver](/yalmip/solvers) available on your path.

````matlab
optimize(Constraints,sum(bound));
````  

The optimal value is, as always, extracted using the overloaded [value](/yalmip/commands/value) command.

````matlab
x_L1 = value(xhat);
````  

The 2-norm problem (least-squares) is easily solved as a QP problem without any constraints.

````matlab
optimize([],residuals'*residuals);
x_L2 = value(xhat);
````

YALMIP automatically detects that the objective is a convex quadratic function, and solves the problem using any installed [QP solver](/yalmip/solvers). If no QP solver is found, the problem is converted to an SOCP, and if no dedicated SOCP solver exist, the SOCP is converted to an SDP.

Finally, we minimize the \\(\infty\\)-norm. This corresponds to minimizing the largest (absolute value) residual. Introduce a scalar to bound the largest value in the vector residual (YALMIP uses MATLAB standard to compare scalars, vectors and matrices)

````matlab
bound = sdpvar(1,1);
Constraints  = [-bound <= residuals <= bound];
````  

and minimize the bound.

````matlab
optimize(Constraints,bound);
x_Linf = value(xhat);
````

We plot the solutions, and notice that the 1-norm estimate worked very well and essentially managed to capture the underlying harmonics despite the severe measurement error after 2 seconds, whereas the two other estimates perform badly on this data set.

````matlab
plot(t,[y A*x_L1 A*x_L2 A*x_Linf]);
````

![Solution to regression problem]({{ site.url }}/images/regresssolution.png){: .center-image }

Let us finally state that some of the manipulations here can be performed much easier by using the [nonlinear operator framework](/tutorial/nonlinearoperator) in YALMIP.

````matlab
optimize([],norm(residuals,1));
optimize([],norm(residuals,inf));
````

### Large-scale quadratic programs

The 2-norm solution is most easily stated in the described QP formulation, although it in some cases is more efficient in YALMIP to express the problem using a 2-norm, which will lead to a [second order cone problem](/tutorial/secondorderconeprogramming).

````matlab
optimize([],norm(residuals,2));
````

The reason is that a quadratic function with \\(n\\) variables can be composed of up to \\(n(n+1)/2\\) monomials, which YALMIP has to work with symbolically. If you absolutely need to solve a large-scale quadratic program with YALMIP using a QP solver, introduce an auxiliary variable and equality constraints. This will make the quadratic term sparse (not only YALMIP but many QP solvers will be significantly faster after this transformation)

````matlab
aux = sdpvar(length(residuals),1);
optimize([aux == residuals],aux'*aux);
````

Of course, in this example, this makes no difference, as there only are 6 decision variables but in scenarios where your objective is \\(x^TQx\\) and \\(Q=R^TR\\) is large, a better model might be

````matlab
R = chol(Q);
z = sdpvar(length(x),1);
optimize([z == R*x],z'*z);
````

Even better, if you know \\(Q\\) is low-rank or there is some other structure that allows you to compute a low-rank possibly sparse factor, you should exploit that

````matlab
R = my_smart_factorization(Q);
z = sdpvar(size(R,2),1);
optimize([z == R*x],z'*z);
````

The archetypical example is **sum(x)^2** which leads to a completely dense quadratic model of rank 1. Absolutely catastrophical for large problems (it will most likely be indefinite for vectors of length larger than 10 in floating point numerics) and a waste of memory. The trivial model is

````matlab
z = sdpvar(1);
optimize([z == sum(x)],z^2);
````
