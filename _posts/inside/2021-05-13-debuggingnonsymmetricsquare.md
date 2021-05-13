---
category: inside
subcategory: 2
excerpt: "Extremely common"
title: Debugging nonsymmetric square warning
tags: [Common mistakes]
date: 2017-09-21
---

One of the most common mistakes found when users face infeasibility or unexpected solutions is unintended definition of linear inequalities instead of semidefinite constraints. This is so common that YALMP now tries to detect these errors and warns about them.

## Incorrect definition of variables

Although it is much more common to define inteded full matrices as symmetric by mistake, sometimes users misunderstand the meaning of full vs symmetric in [sdpvar](/command/sdpvar). Hence a typical mistake might be to try to find a positive definite matrix \\(P\\) proving stability of a linear system by finding a feasible solution to the [semidefinite programming problem](/tutorial/semidefiniteprogramming) \\(A^TP + PA\preceq 0, P\\succeq I, P = P^T\\).

Here we look at the constraint and see that we do not have 2 semidefinite constraints but two sets of elementwise constraints of dimension 4. A warning will be issued for both constraints. In the end, the solver will fail as this linear program is infeasible.

````
A = [-1 2;3 -4];
P = sdpvar(2,2,'full') %WRONG
Model = [A'*P + P*A <= 0, P >= eye(2), P == P']
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
|   ID|                    Constraint|   Coefficient range|
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
|   #1|   Element-wise inequality 4x1|              2 to 8|
|   #2|   Element-wise inequality 4x1|              1 to 1|
|   #3|       Equality constraint 2x2|              1 to 1|
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

optimize(Model)
````

The correct approach is to define a structurally symmetric matrix \\(P\\)
````
P = sdpvar(2,2)
Model = [A'*P + P*A <= 0, P >= eye(2)]
+++++++++++++++++++++++++++++++++++++++++++++++++++++
|   ID|              Constraint|   Coefficient range|
+++++++++++++++++++++++++++++++++++++++++++++++++++++
|   #1|   Matrix inequality 2x2|              2 to 8|
|   #2|   Matrix inequality 2x2|              1 to 1|
+++++++++++++++++++++++++++++++++++++++++++++++++++++
````

## Mistakes in definition of constraints

The most common mistake is some minor mistake with a misplaed transpose or smiliar in the generation of a constraint. Consider once again a control problem where we want to find a positive definite \\(P\\) with \\( \begin{pmatrix} A^TP + PA & PB\\B^TP & -\gamma I \end{pmatrix} \preceq 0\\)




## Bad data

Sometimes the model is correctly setup, the variables are correctly defined, but YALMIP still thinks the obviously symmetric matrix is non-symmetric and issues a warning about a full matrix being used in a square constraint.

The typical cause then is numerical issues where floating-point limitations are causing small errors in computations causing a theoretically symmetric matrix to become non-symmetric in practice. For this to happen, you model has to involve very bad data, and this is an issue you should adress first.

We might have something like this

````
Z = A'*P+P*A
Linear matrix variable 20x20 (full, real, 210 variables)
Coeffiecient range: 1.7462e-10 to 2763983.0299
````

If we try to use this matrix in a constraint \\(Z\succeq 0\\) warnings about a full matrix in a square constraint will appear. To see that there are small terms causing the non-symmetry, we can look at the distance to symmetry

````
Z-Z'
Linear matrix variable 20x20 (symmetric, real, 210 variables)
Coeffiecient range: 1.7462e-10 to 8.6512e-10
````

To circumvent this, you should treat the root-cause which is bad data which most likely will cause issues in the solver too, but a quick fix for the small noise terms is to symmetrize the matrix which hopefully will cancel the small terms

````matlab
Z = (Z + Z');
````

## It is not a mistake!

Sometimes you want to add elementwise constraints on a square full matrix. To avoid this warning you have to make it into a vector constraint such as

````matlab
Model = [A(:) >= B(:)]
````

or 

````matlab
Z = A-B;
Model = [Z(:)]
````

Alternatively, if you think YALMIP is too clever for you, and you want to keep your code as it is, you can turn off the warning (not recommended)


````matlab
warning('off','YALMIP:SuspectNonSymmetry');
````






vector


