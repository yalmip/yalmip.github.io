---
category: inside
subcategory: 2
excerpt: "...but I won't do that."
title: Debugging model creation failed
tags: [Common mistakes]
date: 2017-09-21
---

YALMIP is a language for [disciplined convex programming](/tutorial/nonlinearoperatorsgraphs), [disciplined nonconvex programming](/tutorial/nonlinearoperatorsmixedinteger), and completely undisciplined [throw-anything-at-the-solver programming](/tutorial/nonlinearoperatorscallback). There are cases where you have to use the right modeling strategy though for YALMIP to accept the model.

## Nonconvex use of graph representations

Many operators in YALMIP are modelled in different ways depending on context. The following model is convex, and YALMIP will derive a [linear programming representation](/tutorial/nonlinearoperatorsgraphs)).

````matlab
A = randn(10,2);y=randn(10,1);

x = sdpvar(2,1);
optimize([norm(x,1) <= 1], norm(A*x - y,1))
````

If we change the constraints slightly the model becomes nonconvex, but YALMIP will not complain as it just changes the modeling approach and derives a [mixed-integer linear programming repesentation](/tutorial/nonlinearoperatorsmixedinteger) of the nonconvex constraint instead.

````matlab
optimize([0.1 <= norm(x,1) <= 1], norm(A*x - y,1))
````

Doing the same thing but with a nonconvex use of a 2-norm instead leads to an error

````matlab
optimize([0.1 <= norm(x) <= 1], norm(A*x - y,1))

    solvertime: 0
       problem: 14
          info: 'Model creation failed (learn to debug)(Only 1- and inf-norms can be used in a nonconvex fashion)'
    yalmiptime: 0.0730
    
````

YALMIP does not have a mixed-integer fallback plan for nonconvex use of the 2-norm which normally is modelled using a second-order cone epigraph. This should be of no surprise as it is not a linear constraint or union of linear constraints, and thus not mixed-integer linear programming representable in the nonconvex case.

To proceed, we have to write the constraints using quadratic constraints instead (the convex constraint can of course be kept, but for simplicity we rewrite both). The result is a nonconvex quadratically constrained program, which can be solved using a local nonlinear solver such as [FMINCON](/solver/fmincon) or (if we are lucky) with a global nonlinear solver such as [BMIBNB](/solver/bmibnb), [BARON](/solver/baron) or [GUROBI](/solver/gurobi).

````matlab
optimize([0.1^2 <= x'*x <= 1], norm(A*x - y,1),sdpsettings('solver','fmincon'))
optimize([0.1^2 <= x'*x <= 1], norm(A*x - y,1),sdpsettings('solver','bmibnb'))
````

Operators where this lack of mixed-integer fallbacks are causing problems most often are non-LP representable [norm](/command/norm) and [sqrt](/command/sqrt). In the case of [sqrt](/command/sqrt) squaring a constraint is typically the solution although one also can go for an [alternative implementation](/squareroots).

There are some linear-programming representable functions which in theory could have mixed-integer linear programming representations (such as [sumk](/command/sumk)), but lack this as.



