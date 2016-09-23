---
title: "Nonlinear operators"
category: tutorial
author_profile: false
level: 4
excerpt: "Working with nonlinear operators in a structured and efficient fashion"
layout: single
header:
tags: [Linear programming representable, Second-order cone programming representable, Semidefinite programming representable, Integer programming representable, Exponential cone programming representable]
header:
  teaser:
sidebar:
  nav: "tutorials"
---

Nonlinear functions and operators, such as [abs](/command/abs), [entropy](/command/entropy) or [sort](/command/sort), are equipped with various properties, which allows YALMIP to, e.g., analyze the optimization problem with respect to convexity, select a suitable way to model the problem, or supply gradients and jacobians to nonlinear solvers. If the problem is convex and suitably structured, YALMIP can sometimes use a [graph representation](/tutorial/nonlinearoperatorsgraphs), while a severly nonlinear or nonconvex problem might require the introduction of [mixed-integer representations] or a [callback approach].

### Graph-based representations

Graph-based implementations model the operators by using additional variables and constraints, so called [epi-](http://en.wikipedia.org/wiki/Epigraph_%28mathematics%29) and [hypo-graphs](http://en.wikipedia.org/wiki/Hypograph_%28mathematics%29). The benefit of this modeling strategy is that YALMIP can derive structured optimization models, such as a linear program or semidefinite, instead of treating the operators as general nonlinearities and use a nonlinear solver. Here, we say that the operator is *conic representable* if these reformulations are possible.

These graph representations are only possible if the expressions satisfy certain convexity or concavity conditions. Hence, YALMIP has to analyze the problem to ensure that the representation actually can be used.

Graph-based implementations are available for a large number of operators, such as the [linear programming representable](/tags/#linear-programming-representable) operators  [min](/command/min), [max](/command/max) and [abs](/command/abs), the [second-order cone representable](/tags/#second-order-cone-programming-representable)  [sqrt](/command/sqrt), [norm](/command/norm) and [geomean](/command/geomean), [semidefinite cone representable](/tags/#semidefinite-programming-representable) [nuclear norm], or [exponential cone representable](/tags/#exponential-programming-representable) such as [entropy](/command/entropy). Adding new operators is easy, and can be done almost entirely from template code.

Read more about [graph models of conic representable function](/tutorial/nonlinearoperatorsgraphs).

### Mixed-integer representations

Mixed-integer representations are models that encode an exact representation of an operator by using integer and binary decision variables. The benefit of using such a representation is that the resulting nonconvex problem typically is a well structured problem, such as a mixed-integer linear program. Many of the conic representable operators that are implemented using linear programming graphs, are also available in a mixed integer representation. If YALMIP fails to propagate convexity, it will switch from graph-based modelling to mixed-integer modelling (unless the option *'allownonconvex'* is set to 0). Mixed-integer models are available for, e.g., [min], [max](/command/max) and [abs](/command/abs), but also for purely combinatorial operators operators and conditions without any graph variants such as [or], [ne], [iff](/command/iff), [implies](/command/implies), [nnz](/command/nnz), [sort](/command/sort) and many more, and on a higher level, piecewise affine and quadratic functions in connection with [MPT](/solver/mpt)(/solver/mpt).

Read more about [integer representable function](/tutorial/nonlinearoperatorsmixedinteger).

### Call-based representations

A third way to model operators in YALMIP is by using simple callback evaluations. This is the way a modelling layer normally works with nonlinear solver. The modelling layer simply creates a framework for computing function values and derivatives. Operators modelled this way in YALMIP can also be equipped with convexity information, and thus fit into YALMIPs convexity propagations, and for use in the built-in global solver [BMIBNB](/solver/bmibnb) they can have more information attached such as envelope approximators, bound generators, and inverse functions.

````matlab
yalmip('clear');
sdpvar x;y = exp(x);
aux = model(y);aux{1}

ans = 

       convexity: 'convex'
    monotonicity: 'increasing'
    definiteness: 'positive'
           model: 'callback'
      convexhull: @convexhull
          bounds: @bounds
      derivative: @(x)exp(x)
         inverse: @(x)log(x)
           range: [0 Inf]
            name: 'exp'
          models: 2
          domain: [-Inf Inf]
````          

Since they are based on callbacks, they can only be used with general purpose nonlinear solvers, such as [FMINCON](/solver/fmincon), [SNOPT](/solver/snopt) or [IPOPT](/solver/ipopt). 

Most of MATLABs built-in nonlinear functions, such as [erf](/command/erf) and [sin](/command/sin)), are available as callback operators in YALMIP. There are also operators such as [exp](/command/exp) and [kullbackleibler](/command/kullbackleibler) which can be used both with graph representations in [exponentialconeprogramming](/tutorial/exponentialconeprogramming) and as callback operators in general nonlinear solvers.

Read more about [callback operators](/tutorial/nonlinearoperatorscallback).
