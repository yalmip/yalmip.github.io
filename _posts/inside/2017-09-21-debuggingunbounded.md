---
category: inside
subcategory: 2
permalink: debuggingunbounded
excerpt: "Where to start?"
title: Debugging unbounded models
tags: [Common mistakes]
date: 2017-09-21
---

As a sister post to [debugging infeasible models](/debugginginfeasible), let us study the case where you are faced with complaints about an unbounded model

````matlab

sol = optimize(Constraints,Objective)

    yalmiptime: 0.2192
    solvertime: 0.2498
          info: 'Unbounded problem (MOSEK)'
       problem: 2
````

Before sending a post to the YALMIP [forum](https://groups.google.com/forum/#!forum/yalmip) to resolve the issue, you always make some minimal initial investigation.

### 1. Is it really unbounded?

To begin with, get rid of the objective function. Unboundedness can only arise due to an objective, but solvers can sometimes get confused due to various primal-dual presolve strategies etc. Hence, solve the problem without an objective. If it shows infeasibility, unboundedness is perhaps not the case (or you have a completely flawed model which is both infeasible and unbounded (without the infeasible constraints), and then you have to sort out the infeasibility first)

````matlab
optimize(Constraints)
    yalmiptime: 0.1859
    solvertime: 0.2381
          info: 'Successfully solved (MOSEK)'
       problem: 0
````

Nope, not that simple...

### 2. Bound the variables

If the solver says it is unbounded, the standard trick is to artificially bound the solution-space and solve the problem. If this new problem is solved without problems, you have a clear indication that the original problem is flawed and unbounded (due to unbounded variables) 

The typical strategy is that you bound the solution-space using a large bounded set (such as a cube, or problem specific by bounding the trace of some positive semidefinite matrix you are working with etc), solve the problem, and then look at the solution to see if any variables end up at the artifially introduced bound. If they do, you have found your unbounded variables

````matlab
sdpvar x y z

Model = [-1 <= [x y] <= 1, z <= 0]
Objective = x^2 + y + z;
optimize(Model,Objective)
ans = 

    yalmiptime: 0.1815
    solvertime: 0.0405
          info: 'Unbounded objective function (QUADPROG)'
       problem: 2
       

optimize([Model,-1000 <= [x y z] <= 1000],Objective)
ans = 

    yalmiptime: 0.1346
    solvertime: 0.0064
          info: 'Successfully solved (QUADPROG)'
       problem: 0

>> value([x y z])

ans =

           0          -1       -1000

````

Clearly, the last variable appears unbounded as it ended up at the artificial bound. Increasing the artificial bounds simply leads to a larger value on the variable, which is used in the objective and thus leads to unboundedness.

In many cases your model is complex with loads of variables, and it might be cumbersome to list and constrain all variables. A convenient trick then is to use [recover](/command/recover) and [depends](/command/depends) to collect all relevant variables

````matlab
UsedInObjective = recover(depends(Objective));
optimize([Model, -1000 <= UsedInObjective <= 1000] ,Objective)
ans = 
>> value(UsedInObjective)

ans =

           0
          -1
       -1000
````

### 3. Numerical nonsense

If you have extremely poor numerical data in your model, the solver might get confused as your objective grows large also with small values on the decision variables.

````matlab
Model = [-1 <= [x y] <= 1, z <= 0]
Objective = x^2 + y*1e12+(z-1)^2*1e18;
optimize(Model,Objective)
````

A model like this is a recipie for disaster, and a clear sign that you have developed your model in the wrong units.
