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

Consider the problem of investigating the optimal value \\(x\\) in the optimization problem \\(\min_x (x-b)^2 s.t -a\leq x \leq a\\), as \\(a\\) and \\(b\\) varies. Creating an [optimizer](/command/optimizer) object for this setup is essentially done in the same fashion as when we solve the optimization problem, except that we declare parameters ( \\(a\\) and \\(b\\)) and outputs (\\(x\\)) as trailing arguments. It is recommended to explicitly declare the solver to be used. Here we specify [MOSEK](/solver/mosek), as we trivially know that the optimization problem will be a quadratic program once \\(a\\) and \\(b\\) are fixed (it is a quadratic program also when they are decision variables).

````matlab
sdpvar x a b
Constraints = [-a <= x <= a];
Objective = (x-b)^2;
Saturation = optimizer([-a <= x <= a], Objective,sdpsettings('solver','mosek'),[a;b],x)
````

If we want to solve the quadratic program for particular values of \\(a=1\\) and \\(b=3\\), we simply call the operator with those values

````matlab
xoptimal = Saturation([1;3])
````

### Improved syntax

If you have used [optimizer](/command/optimizer) before, you might notice that we now call the object with parantheses instead of curly brackets. Parantheses is the new standard, but curly brackets still apply.

````matlab
xoptimal = Saturation{[1;3]}
````
Instead of vectorizing the parameters into one vector, we can, as before, create an [optimizer](/command/optimizer) object using multiple input arguments by placing these in a cell in the declaration.

````matlab
Saturation = optimizer([-a <= x <= a], Objective,sdpsettings('solver','mosek'), { a , b }, x)
````

To be backwards compatible and be somewhat lax in common confusions between the old format (first line below) and prefered new (two last), the solution can be obtained with

````matlab
xoptimal = Saturation{ {1,3} }
xoptimal = Saturation{  1,3 }
xoptimal = Saturation( {1,3} )
xoptimal = Saturation( 1 , 3 )
````

This method to call the object is the fastest, but if you are ok with sacrificing some performance, you can use the following format (which is somewhat slower as it requires definitions and analysis of constraint objects). Note that the order does not make a difference.

````matlab
xoptimal = Saturation(b == 3, b == 1)
xoptimal = Saturation([a == 1, b == 1])
````

The variables have to be specified in exactly the same form as they were declared in the creation of the object. Hence, the following will not work

````matlab
xoptimal = Saturation([a;b] == [1;3])
````


### Partial instantiation

The largest change in the [optimizer](/command/optimizer) framework is the introduction of partial instantiation. This means you can create an object where only a subset of parameters have been fixed.

````matlab
Saturation_fixed_a = Saturation(1,[])
Saturation_fixed_a = Saturation([ a == 1])
Saturation_fixed_b = Saturation([],3)
Saturation_fixed_b = Saturation([ b == 3])
````

These objects can now be manipulated further, as it is just an [optimizer](/command/optimizer) object with a smaller amount of parameters.

````matlab
Saturation_fixed_b = Saturation([],3)
x_optmal = Saturation_fixed_b{1}
````

This generalizes to more variables of course

````matlab
sdpvar c d
Saturation = optimizer([-a - c - d <= x <= a + c + d], Objective,sdpsettings('solver','mosek'), { a , b , [c;d]}, x)
Saturation_fixed_ac = Saturation{[a == 1, [c;d] == 0]};
Saturation_fixed_ac = Saturation{1,[],[0;0]};
xoptimal = Saturation_fixed_ac{3}

````




