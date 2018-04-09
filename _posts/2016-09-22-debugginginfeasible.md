---
layout: single
excerpt: "Where to start?"
title: Debugging infeasible models
tags: [Typical problems, Tricks]
comments: true
date: '2016-09-22'
---

You've created your massive 5000 lines of code model, and when you run it, the solver claims it is infeasible

````matlab

sol = optimize(Constraints,Objective)

    yalmiptime: 0.2192
    solvertime: 0.2498
          info: 'Infeasible problem (MOSEK)'
       problem: 1
````

Where to start...

Before asking a question on the YALMIP [forum](/https://groups.google.com/forum/#!forum/yalmip), make sure you've at least covered the first four tips here.

### 1. Absolutely most common mistake

Code looks like this

````matlab
x = sdpvar(n,m);
````

Works like a charm as long as **n** and **m** are different, but when they are equal, you most likely don't want a symmetric matrix which this will create. Hence, you should have had

````matlab
x = sdpvar(n,m,'full');
````


### 2. Is it really infeasible?

To begin with, get rid of the objective function. An objective function cannot generate any infeasibility, but in the feasibility analysis, it is just unnecessary to keep it. You might have stumbled into a bug in the solver presolve code or something, which causes it to make an incorrect statement. Some solvers mess up infeasibility with unbounded objective. If that is the case, you would have to [debug your unbounded model](/debuggingunbounded). Hence, if the problem without objective is feasible, the problem is in the objective and not in the constraints

````matlab
optimize(Constraints)
    yalmiptime: 0.1859
    solvertime: 0.2381
          info: 'Infeasible problem (MOSEK)'
       problem: 1
````

Nope, not that simple...

### 3. Get a second opinion

Solvers can fail, so try another solver.

````matlab
optimize(Constraints,[],sdpsettings('solver','gurobi'))
    yalmiptime: 0.3514
    solvertime: 0.2166
          info: 'Infeasible problem (GUROBI)'
       problem: 1
````

OK, unlikely that two solvers make the same incorrect judgement.

### 4. Clean up and simplify your model

Searching for a needle is easier in a clean small room, than a messy huge room. You don't have to debug your complete model, if the feasibility remains when you remove most parts of it. Make a quick effort to remove stuff. You might find the bug by simply looking at the condensed code...

### 5. Do you have a known feasible solution?

If you have a known feasible solution, use that and see if your model actually is feasible when you use it. Simply assign the solution and check the constraints

````matlab
assign(x,claimedfeasible);
check(Constraints)
+++++++++++++++++++++++++++++++++++++++++++++++++++++
|    ID|               Constraint|   Primal residual|
+++++++++++++++++++++++++++++++++++++++++++++++++++++
|    #1|   Elementwise inequality|                 1|
|    #2|   Elementwise inequality|                 0|
|    #3|   Elementwise inequality|                 1|
|    #4|   Elementwise inequality|                 0|
|    #5|   Elementwise inequality|                -1|
|    #6|   Elementwise inequality|                 0|
|    #7|   Elementwise inequality|                 1|
|    #8|   Elementwise inequality|                 0|
|    #9|   Elementwise inequality|                 1|
|   #10|   Elementwise inequality|                 0|
|   #11|      Equality constraint|                 0|
+++++++++++++++++++++++++++++++++++++++++++++++++++++
````

If this would have shown all constraints feasible, you would have found a bug in both YALMIP and all solvers you've tested. Most likely it will show that some constrant is infeasible, as in this case where contraint 5 is violated.

So, constraint 5? With the set of constraints listed above, it might be a nightmare to figure out which constraint this actually is. This is where [tagging constraints](/taggingconstraints) might help. Add some nice tags in your code that defines the constraints, and it might look like this instead

````matlab
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
|    ID|               Constraint|   Primal residual|                   Tag|
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
|    #1|   Elementwise inequality|                 1|   Fubar constraints 1|
|    #2|   Elementwise inequality|                 0|     Foo constraints 1|
|    #3|   Elementwise inequality|                 1|   Fubar constraints 2|
|    #4|   Elementwise inequality|                 0|     Foo constraints 2|
|    #5|   Elementwise inequality|                -1|   Fubar constraints 3|
|    #6|   Elementwise inequality|                 0|     Foo constraints 3|
|    #7|   Elementwise inequality|                 1|   Fubar constraints 4|
|    #8|   Elementwise inequality|                 0|     Foo constraints 4|
|    #9|   Elementwise inequality|                 1|   Fubar constraints 5|
|   #10|   Elementwise inequality|                 0|     Foo constraints 5|
|   #11|      Equality constraint|                 0|                      |
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
````

Hence, the *Fubar* constraint you setup in iteration 3, is not correct, or at least not consistent with the solution that you claim is valid.

An alternative is to modularize your code a bit and create sub-components

````
% Create The Fubar constraints
Fubar = ...

% Create The Foo constraints
Foo = ...

optimize([Fubar, Foo])
check(Fubar)
check(Foo)
````

### 6. Solve a model with slacked constraints

Try to add slacks on constraints, and minimize the slack. Very often, you will see non-zero slacks just a few constraints, and they are often the guilty ones. At least it helps you to hone in on problematic constraints.

Hence, we replace

````matlab
Constraints = []
for i = 1:N
 Constraints = [Constraints, something1 <= 0];
 Constraints = [Constraints, something2 == 0];
end
````

with some slacked variant, such as

````matlab
slack1 = sdpvar(N,1);
slack2 = sdpvar(N,1);
Constraints = [slack1>=0]
for i = 1:N
 Constraints = [Constraints, something1 <= slack1(i)];
 Constraints = [Constraints, something2 == slack2(i)];
end
````

and solve the problem while trying to drive the slacks to zero

````matlab
optimize(Constraints, sum(slack1) + sum(abs(slack2))
````

Checking the values of the slacks could reveal something

````matlab
value(slack1)
ans =
  0  0  0   0  0.5000
value(slack2)
ans =
  0  0  0   0  0.5000
  
````

The fifth constraints combined in the two sets of constraints appear to be problematic, as we cannot find a solution where they both are feasible.


### 7. Bisect you constraints

Remove constraints, and see when it becomes feasible. In the end, this might be your only option to hone in on the problems in your code. You can do this either by commenting out parts in your code, or by indexing from the full set.

In the following example, we have a model with 11 constraints which is infeasible

````matlab
sol = optimize(Constraints(1:5));if sol.problem==0;display('Feasible');else;display('Infeasible');end
Feasible
sol = optimize(Constraints(6:11));if sol.problem==0;display('Feasible');else;display('Infeasible');end
Feasible
````

Nasty, there is a combination of some of the first 5 constraints (which are feasible on their own) which combined with the last 6 constraints (which are feasible on their own) causing infeasibility.

````matlab
sol = optimize(Constraints(1:8));if sol.problem==0;display('Feasible');else;display('Infeasible');end
Feasible
sol = optimize(Constraints(1:10));if sol.problem==0;display('Feasible');else;display('Infeasible');end
Feasible
````

From this we know that the problem occurs when the eleventh constraint is added to the model, in combination with the other. Now you just have to come up with strategies to dig further. Essentially some kind of bisection.

````matlab
sol = optimize(Constraints([1:5 11]));if sol.problem==0;display('Feasible');else;display('Infeasible');end
Infeasible
sol = optimize(Constraints([1:3 11]));if sol.problem==0;display('Feasible');else;display('Infeasible');end
Feasible
sol = optimize(Constraints([4:5 11]));if sol.problem==0;display('Feasible');else;display('Infeasible');end
Infeasible
sol = optimize(Constraints([5 11]));if sol.problem==0;display('Feasible');else;display('Infeasible');end
Infeasible
````

There we have it. Constraint 5 and 11 are inconsistent. Figure out why!

