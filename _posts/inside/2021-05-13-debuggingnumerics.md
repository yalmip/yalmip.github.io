---
category: inside
subcategory: 2
excerpt: "Crap in crap out"
title: Debugging numerical problems
tags: []
date: 2017-09-21
---

A very common problem faed when solving problem is complaints in the solver about numerical problems, or other diagnostics codes which indicate problems solving the problem. There are many possble causes for this, from benign issues causing minor warnings in the solver, to completely disastrous problems in the model.

## Crap in crap out - bad data

Solvers work in finite precsion in floating-point numerics, and this means most computations all the way down to addition and substraction, only is an approximation. As the solver progress, these small errors add up, and in some models with bad data this can lead to failure.

So what is bad data? As the saying goes you know it when you see it. There is no clear-cut definition of bad data in a model. Large numbers and very small numbers are typically the root cause, in particular when the model contains both, as a scaling strategy then can be hard to apply for the solver. So that leads to the question what are small and large? One again ,there is no strict definition. Simply speaking, the larger spread in the order of magnitudes among coefficients the worse. In other words, for non-zero numbers, the further away from 0, in a logarithmic measure, the worse. You can typically start to expect issues wen you go below \\(10^{-6}\\) or above \\(10^6\\) or so. Once again though, this is not a certain fact. In a lucky situation your solver might work very well with data in the order of \\(10^12\\) but then another day it fails on data with muh smaller coefficients.

There is no simple trick to fix generic bad data. You simply have to understand why your model contains bad data, and try to define a better model. 

A typical issue might be that you re working inthe wrong units. You are planning a trip to Mrs and measureing distance in meters instead of kilometers, or you are computing energies in atoms and express distances in meters  instead of nanometers. In control theory, a common cause is badly conditoned state-space realization. A way to obtain better data then can be to perform a balanced realization before defining the model.

### Debuging and remedy

There is no simple trick to fix generic bad data. You simply have to understand why your model contains bad data, and try to define a better model. 

A typical issue might be that you re working inthe wrong units. You are planning a trip to Mrs and measureing distance in meters instead of kilometers, or you are computing energies in atoms and express distances in meters  instead of nanometers. In control theory, a common cause is badly conditoned state-space realization. A way to obtain better data then can be to perform a balanced realization before defining the model.

To debug this issue, you have to take a good look at your data and the process generating your data. To see where you have bad data in your model, you can display objects and they list the smallest and largest (absolute) coefficients.

````matlab
sdpvar x y z
E = 1e-8*x + y + 1e8*z
Linear scalar (real, 3 variables)
Coefficients range: 1e-08 to 100000000

Model = [E == 5e12]
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
|   ID|                Constraint|        Coefficient range|
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
|   #1|   Equality constraint 1x1|   1e-08 to 5000000000000|
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
````

## Ill-posed problems

A second category of issues arise in ill-posed problems. A simple example could be minimizing \\(x^{-1}\\) on \\(x\geq 0\\). A solver might run into troubles as the iterate \\(x\\) will diverge to infinity. Thi is a very common situation in control theory where optimal state-feedback solutions can involve controllers which force some poles to \\(-\infty\\), which requires some decision variables growing arbitrarily large.

### Debugging and remedy

Both debugging and the remedy can be done by adding bounds or penalties on variables. 

To debug this issue, simply add a large bound (but not so large that it cuses issues with bad data) and study the solution to see if any variable appear to approach infinity. If some variable ends up t the bound, no matter what you set the bound to, you have most likely found the issue.

````matlab
sdpvar x
optimize([1000 >= x >= 0],1/x);
value(x)

ans =

   1.0000e+03
````

In a more complex scenario, the command [allvariables](/command/allvariables) can be convenient in a first test

````matlab
optimize([Model, -10000 <= allvariables(Model,Objective) <= 10000],Objective)
value(allvariables(Model,Objective) )
````

Having found the issue, you should first try to understand why some variables tend to infinity, and if it makes sense. Maybe you have missed some constraints. Although it is optimal, is it good? Coming back to the control example, the solution at infinity is optimal, but it is very fragile and not practically relevant. Hence, you might want to add constraints on some variables,or add some penalty on some or all variables to avoid this prolematic escape to infinity

````matlab
optimize(Model,Objective + 0.1*norm(allvariables(Model,Objective)))
````


### Non-strictly feasible solutions

A common scenario is that yoo define a problem and then replace a non-trict onstraint with strict constraint by adding some small margin to a constraint. If the problem lacks a strict solution, and you have added a margin which is so small that it drowns in the general tolerances of the solver, the solver can easily run into numerical problems as it thinks it is very close to solve the problem, but struggles on the last bit (naturally, as the robolem is infeasible).

### Infimizers

A tricky issue can arise if you have a nonlinear model where the minimizer isn't attainable.


As a sister post to [debugging infeasible models](/debugginginfeasible), let us study the case where you are faced with complaints about an unbounded model

````matlab

sol = optimize(Constraints,Objective)


    yalmipversion: '20210331'
    matlabversion: '9.9.0.1524771 (R2020b) Update 2'
       yalmiptime: 2.220583525175668e-01
       solvertime: 7.194164748243008e-02
             info: 'Unbounded objective function (learn to debug)(GUROBI-GUROBI)'
          problem: 2
````

or even worse, the solver cannot understand if it is unbounded, infeasible, or perhaps even both

````matlab
    yalmipversion: '20210331'
    matlabversion: '9.9.0.1524771 (R2020b) Update 2'
       yalmiptime: 2.220583525175668e-01
       solvertime: 7.194164748243008e-02
             info: 'Either infeasible or unbounded (learn to debug)(GUROBI-GUROBI)'
          problem: 12
````          

Before sending a post to the YALMIP [forum](https://groups.google.com/forum/#!forum/yalmip) to resolve the issue, you always make some minimal initial investigation.

### 1. Is it really unbounded?

To begin with, get rid of the objective function. Unboundedness can only arise due to an objective, but solvers can sometimes get confused due to various primal-dual presolve strategies etc. Hence, solve the problem without an objective. If it shows infeasibility, unboundedness is perhaps not the case (or you have a completely flawed model which is both infeasible and unbounded (without the infeasible constraints). If infeasibility is detected, you have to [sort out the infeasibility first](/debugginginfeasible).

````matlab
    yalmipversion: '20210331'
    matlabversion: '9.9.0.1524771 (R2020b) Update 2'
       yalmiptime: 7.407308661948966e-02
       solvertime: 1.926913380510843e-03
             info: 'Successfully solved (GUROBI-GUROBI)'
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
    yalmipversion: '20210331'
    matlabversion: '9.9.0.1524771 (R2020b) Update 2'
       yalmiptime: 2.157324445228372e-01
       solvertime: 1.302675554771629e-01
             info: 'Either infeasible or unbounded (learn to debug)(GUROBI-GUROBI)'
          problem: 12
       

optimize([Model,-1000 <= [x y z] <= 1000],Objective)
ans = 


    yalmipversion: '20210331'
    matlabversion: '9.9.0.1524771 (R2020b) Update 2'
       yalmiptime: 2.437561346474919e-01
       solvertime: 1.402438653525084e-01
             info: 'Successfully solved (GUROBI-GUROBI)'
          problem: 0

>> value([x y z])

ans =

           0          -1       -1000

````

Clearly, the last variable appears unbounded as it ended up at the artificial bound. Increasing the artificial bounds simply leads to a larger value on the variable, which is used in the objective and thus leads to unboundedness.

In many cases your model is complex with loads of variables, and it might be cumbersome to list and constrain all variables. A convenient trick then is to use [allvariables](/command/allvariables) to collect all involved variables

````matlab
UsedInObjective = allvariables(Objective);
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
