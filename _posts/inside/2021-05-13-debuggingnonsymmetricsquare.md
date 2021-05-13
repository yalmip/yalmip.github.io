---
category: inside
subcategory: 2
excerpt: "Extremely common"
title: Debugging nonsymmetric square warning
tags: [Common mistakes]
date: 2017-09-21
---

One of the most common mistakes found when users face infeasibility or unexpected solutions is unintended definition of linear inequalities instead of semidefinite constraints. This is so common that YALMP now tries to detect this mistake to warn about it.

## Incorrect definition of variables

Although it is much more common to define intended full matrices as symmetric by mistake, sometimes users misunderstand the meaning of *full* vs *symmetric* in [sdpvar](/command/sdpvar). A typical mistake could be to try to prove stability of a linear system by finding a solution to the [semidefinite programming problem](/tutorial/semidefiniteprogramming) \\(A^TP + PA\preceq 0,~P\\succeq I,~P = P^T\\).

We create this model incorrectly, and note when displaying the constraint (always do that in the initial phase of your model development!) that we do not have 2 semidefinite constraints but two sets of elementwise constraints of dimension 4. A warning will be issued for both constraints. In the end, the solver will fail as this *linear program* is infeasible.

````matlab
A = [-4 2;3 -4];
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

The [correct approach](/tutorial/basics) is to define a structurally symmetric matrix \\(P\\). Since we now have semidefinite constraints a semidefinite programming solver will be called and easily solve the problem.

````matlab
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

The most common reason is some minor mistake with a misplaced transpose or similiar in the construction of a constraint. Consider once again a control problem where we want to find a positive definite \\(P \succeq I\\) with \\( \left( \begin{array} A^T P + PA & PB\\B^TP & 1-\gamma \end{array} \right) \preceq 0\\).

Since we are prone to make mistakes, we display the constraint object which correctly gives us a warning when we define it. The code below contains a mistake which turns the intended semidefinite constraint into 9 elementwise constraints. Trying to solve this model leads to infeasibility.

````matlab
A = [-4 2;3 -4];B = [1;1];
P = sdpvar(2,2);
sdpvar gamma

M = [A*P + P*A P*B;B'*P 1-gamma];
Model = [P >= 0, M <= 0]
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
|   ID|                    Constraint|   Coefficient range|
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
|   #1|         Matrix inequality 2x2|              1 to 1|
|   #2|   Element-wise inequality 9x1|              1 to 8|
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

optimize(Model)
````

Although it is easy to spot the mistake here (right!) we need a strategy in the general case. The warning will be issued when we create the Model object, so it is not obvious which of the two constraints YALMIP deems suspcious. Define them separately to see this, or look at the generated constraint which lists the second constraint as an elementwise constraint instead of the intended semidefinite constraints. 

We thus know the matrix \\(M\\) accidentally has beome non-symmetric. To find the mistake in this matrix, it is convenient to use [spy](/command/spy) which shows non-zero elements in a matrix. Without any output, it gives a graphical view. Alternatively catch the output and display it. \\(M\\) is supposed to be symmetric, so check this

````matlab
s = full(spy(M - M'))

s =

  3×3 logical array

   0   1   0
   1   0   0
   0   0   0

````

The upper left block is not symmetric, and we can hone in on \\( A^TP + PA\\) to find the missing tranpose on \\A\\).


## Bad data

Sometimes the model is correctly constructed, the variables are correctly defined, but YALMIP still thinks the obviously symmetric matrix is non-symmetric and issues a warning about a full matrix being used in a square constraint.

The typical cause then is numerical issues where floating-point limitations are causing small errors in computations causing a theoretically symmetric matrix to become non-symmetric in practice. For this to happen, you model has to involve very bad data, and [this is an issue you should adress first](/inside/debuggingnumerics)

We might have something like this

````matlab
Z = A'*P+P*A
Linear matrix variable 20x20 (full, real, 210 variables)
Coeffiecient range: 1.7462e-10 to 2763983.0299
````

If we try to use this matrix in a constraint \\(Z\succeq 0\\) warnings about a full matrix in a square constraint will rightfully appear. To see the small terms causing the non-symmetry, we can look at the distance to symmetry

````matlab
Z-Z'
Linear matrix variable 20x20 (symmetric, real, 210 variables)
Coeffiecient range: 1.7462e-10 to 8.6512e-10
````

Just as above, we can look at the pattern of \\(Z-Z^T\\) which in theory should be 0

````matlab
spy(Z-Z')
````

To circumvent this, you should treat the root-cause with bad data which most likely will cause issues in the solver too, but a quick fix for the noise terms is to symmetrize the matrix which hopefully will cancel the small terms

````matlab
Z = (Z + Z');
````

## It is not a mistake!

Sometimes you want to add elementwise constraints on a square full matrix. To avoid this warning you have to make it into a non-square constraint such as

````matlab
Model = [A(:) >= B(:)]
````

or 

````matlab
Z = A-B;
Model = [Z(:) >= 0]
````

Alternatively, if you think YALMIP is too clever, and you want to keep your code as it is, you can turn off the warning (not recommended)

````matlab
warning('off','YALMIP:SuspectNonSymmetry');
````
