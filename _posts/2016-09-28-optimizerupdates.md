---
layout: single
excerpt: Slice'n dice your problems"
title: "Extensions on the optimizer"
tags: 
comments: true
features: false
date: '2016-09-28'
---


The [optimizer](/command/optimizer) has been revamped significantly internally, and comes with new features and simplified use.

For those of you who have missed this feature, [optimizer](/command/optimizer) allows you to define a parameterized optimization problem which YALMIP precompiles to its internal numerical format. This object can then be instantiated for particular values on the parameters. This allows for, e.g., rapid simulation and experimentation with optimization problems, as most of the YLMIP overhead is removed from the loop.

Consider the problem of investigating the optimal value \\(x\\) in the optimization problem \\(\min_x (x-b)^2 s.t -a\leq x \leq a\\), as \\(a\\) and \\(b\\) varies. Creating an [optimizer](/command/optimizer) object for this setup is essentially done in the same fashion as when we solve the optimization problem, except that we declare parameters ( \\(a\\) and \\(b\\)) and outputs (\\(x\\)) as trailing arguments. It is recommended to explicitly declare the solver to be used. Here we specify [MOSEK](/solver/mosek), as we trivially know that the optimization problem will be a quadratic program once \\(a\\) and \\(b\\) are fixed (it is a quadratic program also when they are decision variables)

````matlab
sdpvar x a b
Constraints = [-a <= x <= a];
Objective = (x-b)^2;
Saturation = optimizer([-a <= x <= a], Objective,sdpsettings('solver','mosek'),[a;b],x)
````


