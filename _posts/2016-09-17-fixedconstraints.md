---
layout: single
excerpt: "Code works for almost all cases, but suddenly fails."
title: Constraints without any variables
tags: [Common problems]
comments: true
date: '2009-08-29'
---

Sometimes when working with complex models, it might happen that some constraints are trivially feasible or infeasible, since they evaluate to a fixed number independent of all decision variables. This can cause problems for YALMIP in some cases.

Consider the following model, where the numbers **k** and **m** are constants in your code

````matlab
sdpvar x y
Constraints = [x >= 1, y >= k*y + m]
optimize(F,obj)
````

This is a perfectly valid model. Now, let us assume that the magic numbers **k** and **m** all of a sudden happen to have the values 1 and 0. This would correspond to the following case

````matlab
k = 1;
m = 1;
sdpvar x y
Constraints = [x >= 1, y >= k*y + m]
optimize(F,obj)
````

When concatenating the constraints, MATLAB will first tell YALMIP to evaluate all the expressions. When doing so, YALMIP changes the expressions to {$x - 1\geq 0$} and {$y-ky-m\geq 0$}. The second expression will simply lead to the expression **0>=0**, i.e., standard MATLAB code without any YALMIP context, which evaluates to **logical(1)**. Hence, the model will be

````matlab
Constraints = [x-1 >= 0, logical(1)]
optimize(F,obj)
````

When encountering such a situation, YALMIP will issue a warning. An even worse case can occur if, e.g., **k=1** and **m = 1**. We then have the model

````matlab
Constraints = [x-1 >= 0, -1 >= 0]
optimize(F,obj)
````
This will issue an error in YALMIP since the model by construction is infeasible.
