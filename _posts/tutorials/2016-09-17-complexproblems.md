---
title: "Complex-valued problems"
category: tutorial
level: 4
tags: [Complex data]
excerpt: "Complex data in optimization models. No problem in reality."
sidebar:
  nav: "tutorials"
---


YALMIP supports complex-valued constraints for all solvers by automatically converting complex-valued problems to real-valued problems.

To begin with, let us just define a simple linear complex problem to illustrate how complex variables and constraints are generated and interpreted.

````matlab
p = sdpvar(1,1,'full','complex');      % A complex scalar (4 arguments necessary)
s = sdpvar(1,1)+sqrt(-1)*sdpvar(1,1);  % Alternative definition
F = [0.9 >= imag(p)];                  % Imaginary part constrained
F = [F, 0.01 >= real(p)];              % Real part constrained
F = [F, 0.1+0.5*sqrt(-1) >= p];        % Both parts constrained
F = [F, s+p == 2+4*sqrt(-1)];          % Both parts constrained
````

To see how complex-valued constraints can be used in a more advanced setting, we solve the covariance estimation problem from the [sedumi](/command/sedumi) manual. The problem is to find a positive-definite Hermitian Toeplitz matrix **Z** such that the Frobenious norm of **P-Z** is minimized (**P** is a given complex matrix.)

The matrix **P** is
````matlab
P = [4 1+2*i 3-i;1-2*i 3.5 0.8+2.3*i;3+i 0.8-2.3*i 4];
````

We define a complex-valued Toeplitz matrix of the corresponding dimension

````matlab
Z = sdpvar(3,3,'toeplitz','complex')
````

A complex Toeplitz matrix is not Hermitian, but we can make it Hermitian if we remove the imaginary part on the diagonal.

````matlab
Z = Z-diag(imag(diag(Z)))*sqrt(-1);
````

Minimizing the Frobenious norm of **P-Z** can be cast as minimizing the Euclidean norm of the vectorized difference **P(:)-Z(:)**. By using a Schur complement, we see that this can be written as the following SDP.

````matlab
e = P(:)-Z(:)
t = sdpvar(1,1);
F = [Z>=0];
F = [F, [t e';e eye(9)]>=0];
optimize(F,t);
````

The problem can be implemented more efficiently using a second order cone constraint.

````matlab
e = Z(:)-P(:)
t = sdpvar(1,1);
F = [Z>=0];
F = [F, cone(e,t)];
optimize(F,t);
````

...with a second order cone constraint that we let YALMIP model automatically

````matlab
e = Z(:)-P(:)
F = [Z>=0];
optimize(F,norm(e,2));
````

...or by using a quadratic objective function

````matlab
e = Z(:)-P(:)
F = [Z>=0];
optimize(F,e'*e);
````

...or by simply using the nonlinear operator framework which supports matrix norms

````matlab
F = [Z>=0];
optimize(F,norm(P-Z,'fro'));
````
