---
layout: single
excerpt: "Solving Sudoku games using YALMIP. Easy and slow model, or complicated and fast."
title: Sudoku solver
tags: [Integer programming]
comments: true
date: '2016-09-16'
header:
  teaser: "sudoku.gif"
image:
  feature: lofberg.jpg
  teaser: lofberg.jpg
  thumb: lofberg.jpg
---


In case you have missed out on the [Sudoku](http://en.wikipedia.org/wiki/Sudoku) hype, the goal is to fill in unspecified elements in a matrix with numbers between 1 to 9, keeping elements in all rows and columns different, and keeping all elements in the 9 3x3 blocks different. Unspecified elements are indicated by zeros here.

````matlab
S = [0,0,1,9,0,0,0,0,8;6,0,0,0,8,5,0,3,0;0,0,7,0,6,0,1,0,0;...
     0,3,4,0,9,0,0,0,0;0,0,0,5,0,4,0,0,0;0,0,0,0,1,0,4,2,0;...
     0,0,5,0,7,0,9,0,0;0,1,0,8,4,0,0,0,7;7,0,0,0,0,9,2,0,0];

ans =
0  0  1  9  0  0  0  0  8
6  0  0  0  8  5  0  3  0
0  0  7  0  6  0  1  0  0
0  3  4  0  9  0  0  0  0
0  0  0  5  0  4  0  0  0
0  0  0  0  1  0  4  2  0
0  0  5  0  7  0  9  0  0
0  1  0  8  4  0  0  0  7
7  0  0  0  0  9  2  0  0
````

### High-level model

In this example, we will first use the logic constraint [alldifferent] to pose and solve this problem. Note that this operator introduces (a lot of) binary variables, hence you need to have an efficient integer linear programming solver installed.

We begin by creating our 9x9 integer decision variable and the basic constraint structure.

````matlab
M = intvar(9,9,'full');

fixed = find(S);
F = [1 <= M <= 9, M(fixed) == S(fixed)];
````

We add the logic constraints using some MATLAB indexing tricks, add some redundant cuts, and solve the problem (The solution time depends highly on your MILP solver. [CPLEX], [GUROBI] and [MOSEK] solve this problem in roughly 0 seconds, while [GLPK] and [LPSOLVE] fail to solve the problem in reasonable time.)

````matlab
for i = 1:3
 for j = 1:3
  block = M((i-1)*3+(1:3),(j-1)*3+(1:3))
  F = [F, alldifferent(block)];
 end
end

for i = 1:9
 F = [F, alldifferent(M(i,:))];
 F = [F, alldifferent(M(:,i))];
end

F = [F, sum(M,1) == 45, sum(M,2) == 45];
optimize(F);
````

Note that this model of the Sudoku game is pretty weak due to the simple implementation of the [alldifferent] operator in YALMIP!

### Binary model

An alternative model can be created by using the support for multi-dimensional [sdpvar] variables. We will use a binary three-dimensional variable **A(i,j,k)** to indicate that element **(i,j)** has value **k**.

This model is much stronger, in the sense that it is easily solved using any MILP solver (even YALMIPs native solver [Solvers.BNB | BNB] solves the problem in no time, indicating how simple this problem actually is). The drawback is that the model is much less intuitive, since it doesn't use the simple [alldifferent] operator, but instead relies on pure binary constraints.

We begin by creating the variable, and define the basic Sudoku constraints (unique values in each row and column)

````matlab
S = [0,0,1,9,0,0,0,0,8;6,0,0,0,8,5,0,3,0;0,0,7,0,6,0,1,0,0;...
     0,3,4,0,9,0,0,0,0;0,0,0,5,0,4,0,0,0;0,0,0,0,1,0,4,2,0;...
     0,0,5,0,7,0,9,0,0;0,1,0,8,4,0,0,0,7;7,0,0,0,0,9,2,0,0];

p = 3;
A = binvar(p^2,p^2,p^2,'full');
F = [sum(A,1) == 1, sum(A,2) == 1, sum(A,3) == 1];
````

Setting up the constraints for each 3x3 block is a bit messier.

````matlab
for m = 1:p
    for n = 1:p
        for k = 1:p^2
            s = sum(sum(A((m-1)*p+(1:p),(n-1)*p+(1:p),k)));             
            F = [F, s == 1];
        end
    end
end
````

Define constraints for the specified elements

````matlab
for i = 1:p^2
    for j = 1:p^2
        if S(i,j)
            F = [F, A(i,j,S(i,j)) == 1];
        end
    end
end
````

Note that the loop alternatively could have been simplified to a vectorized expression.

````matlab
[i,j,k] = find(S);
F = [F, A(sub2ind([p^2 p^2 p^2],i,j,k)) == 1];
````

We are ready to invoke the solver.

````matlab
diagnostics = optimize(F);
````

The integer solution is finally recovered from the binary indicators.

````matlab
Z = 0;
for i = 1:p^2
      Z = Z  + i*value(A(:,:,i));
end
Z
````

The loop can alternatively be vectorized.

````matlab
Z = value(reshape(A,p^2,p^4)*kron((1:p^2)',eye(p^2)))
````
