---
layout: single
permalink: /cutsdpsolver
excerpt: "A little known solver"
title: "The CUTSDP solver explained"
tags: [Semidefinite programming, Mixed-integer conic programming solver]
comments: true
date: '2017-08-25'
---

YALMIP is shipped with two mixed-integer conic programming solvers, both relying external solvers to do the heavy work, but in completely different ways.

The most known solver is the solver [bnb](/solver/bnb), which is a classical branch-and-bound implementation for mixed-integer convex programming (it can applied to any problem, but global optimality guarantees only hold for problems where the continuous relaxations are convex). Without a doubt, the solver is terribly slow, and is only intended to be a tool for completing academic proof-of-conceept projects where a mixed-integer semidefinite program is solved. For all other problem classes, there are much better [external solvers](/tags/#mixed-integer-programming-solver).

A less known internal mixed-integer solver is [cutsdp](/solver/cutsdp). While [bnb](/solver/bnb) is based on branch-and-bound where, roughly speaking, integrality is approximated during the iterative process, [cutsdp](/solver/cutsdp) is based on an iterative process where the geometry of the semidefinite cone is approximated during the process, while integrality is kept throughout.

## Cutting planes - from the semidefinite cone to linear elementwise cone

[cutsdp](/solver/cutsdp) is not only a solver for mixed-integer conic programs, but is a general solver for semidefinite programs also in the continuous case (although one would never use it as such). 

The basic idea is simple, and is a classical approach to nonlinear programming. A semidefinite constraint $X\succeq 0$ is by definition equivalent to the infinite-dimensional linear programming model $v^TXv \geq 0 ~ \forall~v$. The trick now is to cleverly generate a lot of vectors $v$ and construct a linear program, such that the geometry of the linear programming model starts approximating the geometry of the semidefinite program. Every such added linear constraint is called a cutting plane. Note that the linear program is an *outer approximation* of the original semidefinite program. This means that a feasible solution to the linear programming approximation is not guaranteed to be a feasible solution to the semidefinite program. It also means that if we are minimizing an objective, it will generate a lower bound.

To see the basic idea in practice, we consider a simple semidefinite constraint defining a ball, and an objective which tells us we want to go as far as possible to the nort-east, and then approximate this model with an increasing number of cutting planes (to speed up the process, we add 20 cutting planes before plotting again. to see a much faster high-level implementation, look at [this post](/randomextension))

````matlab
sdpvar x y
% Manual SDP model of x^2 + y^2 <= 1
X = [1 x y;[x;y] eye(2)];
Objective = -x - y;
ops = sdpsettings('plot.shade',.1,'verbose',0);
% Initial outer approximation
ballApproximation = [-1 <= [x y] <= 1];
clf;hold on
for k = 1:10
  vi = randn(3,1);
  for i = 1:20
    ballApproximation = [ballApproximation, vi'*X*vi >= 0];  
  end
  plot(ballApproximation,[x;y],'blue',200,ops); 
  optimize(ballApproximation,Objective,ops);
  plot(value(x),value(y),'k*');drawnow
  drawnow
end 
````

Creating a solver based on this strategy is primarily about generating relavant cutting planes, instead of randomly placing them everywhere. Since we have an objective function, where are not interested in approximating the whole feasible set, but only need a good approximation around the optimal point. Additionally, as it is in the implementation above, we are generating new cuts which might be completely redundant, i.e., the do not cut away any infeasible points.

Consider a solution leading to a matrix \\(X^{\star}\\). If this solution is infeasible, we know that the smallest eigenvalue of \\(X^{\star}\\) is negative. Hence there is a negative \\(\lambda\\) and a vector  \\(v\\) such that  \\(X^{\star}v = \lambda v\\),. i.e., \\(v^TX^{\star}v = \lambda v^v < 0\). In other words, the current solution vialates the constraint \\(v^TXv \geq 0\\). This indicates that  eigenvectors \\(v\\) associated with negative eigenvalues for the current semidefinite constraints are suitable candidates for creating cutting planes.

````matlab
ballApproximation = [-1 <= [x y] <= 1];
clf;hold on
for k = 1:10  
  optimize(ballApproximation,Objective,ops);
  [V,D] = eig(value(X));  
  vi = V(:,1);
  ballApproximation = [ballApproximation, vi'*X*vi >= 0];    
  plot(ballApproximation,[x;y],'blue',200,ops);   
  plot(value(x),value(y),'k*');drawnow
  drawnow
end 
````

In a very few steps, the semidefinite constraint is perfectly approximated around the true optimal solution, and the problem is solved. Of course, this particular problem is trivial, and for real problems the number of cutting planes can grow very large while still having large infeasibility in the approximation.

The [cutsdp](/solver/cutsdp) implements precisely this strategy, generalized to multiple constraints, and second-order cone constraints.

## Adding integrality constraints

If we now add integrality constraints to the model, nothing really changes. We are still outer approximating the semidefinite cone, but instead of solving linear programs, we will solve mixed-integer linear programs. If the solution to the mixed-integer program satisfies the original semidefinite program, it is our sought solution. If not, it must vialote some semidefinite constraint, and we can add a cutting plane based on a negative eigenvalue. Also note that the feasible set of a purely integer semidefinite program, is a mixed-integer linear program in disguise. The feasible set is the integer lattice points, and the convex hull of these is a polytope.

Let us create a mixed-integer semidefinite program, which models a problem where we are in one of two half-moons,  which can be cast as a mixed-integer semidefinite program (in practice you would simply write it using a quadratic constraint and YALMIP would derive a mixed-integer second-order cone problem instead). Note that you must have an efficient [mixed-integer linear programming solver] installed. The integrality in the model comes from the use of the combinatorial [implications](/command/implies) and the binary variables which defines which half-moon we are in.

````matlab
clf
hold on
X = [1 x y;[x;y] eye(2)];
binvar d
plot([X >= 0, -1 <= [x y] <= 1, implies(d,x>=0.5),implies(1-d,x <= -0.5)]);
Model = [-1 <= [x y] <= 1, implies(d,x>=0.5),implies(1-d,x <= -0.5)];
for i = 1:10
    plot(Model,[x;y],'yellow')
    optimize(Model,-x-y);
    plot(value(x),value(y),'k*');
    [V,D] = eig(value(X))
    v = V(:,1);
    Model = [Model, v'*X*v >= 0];
end
````

Applying exactly the same outer approximation stragy, the optimal solution is found easily.


A direct YALMIP implementation to solve this problem would be

````matlab
plot([X >= 0, -1 <= [x y] <= 1, implies(d,x>=0.5),implies(1-d,x <= -0.5)]);
Model = [X >= 0,
        -1 <= [x y] <= 1, 
        implies(d,x>=0.5),implies(1-d,x <= -0.5)];
optimize(Model, Objective, sdpsettings('solver','cutsdp'));        
````

Through some initial preprocessing, the optimal solution is found already in the first mixed-integer linear program, i.e, the first mixed-integer linear program returns a solution which is feasible in the original mixed-integer semidefinite program, and thus an optimal solution to the original problem.


## Should I use [bnb](/solver/bnb) or [cutsdp](/solver/cutsdp)

It really depends on the problem. [bnb](/solver/bnb)  relaxes integrality but handles the semidfinite geometry exactly, while  [cutsdp](/solver/cutsdp)  relaxes the semidefinite cone while it handles integrality exactly in every iteration. Depending on the geometry of your problem, they will behave differently. Always test both.



