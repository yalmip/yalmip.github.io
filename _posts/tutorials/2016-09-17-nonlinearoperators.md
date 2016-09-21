---
title: "Nonlinear operators"
category: tutorial
author_profile: false
level: 4
excerpt: "Working with nonlinear operators in a structured and efficient fashion"
layout: single
header:
tags: [Conic representable, Integer representable]
header:
  teaser:
sidebar:
  nav: "tutorials"
---

Nonlinear functions and operators, such as [abs], [entropy] or [sort], are equipped with various properties, which allows YALMIP to, e.g., analyze the optimization problem with respect to convexity, select a suitable way to model the problem, or supply gradients and jacobians to nonlinear solvers. If the problem is convex and conic, YALMIP can sometimes use a [graph representation], while a nonconvex problem might require the introduction of [mixed-integer representations] or a [callback approach].

### Graph-based representations

Graph-based implementations model the operators by using additional variables and constraints, so called [epi-](http://en.wikipedia.org/wiki/Epigraph_%28mathematics%29) and [hypo-graphs](http://en.wikipedia.org/wiki/Hypograph_%28mathematics%29). The benefit of this modeling strategy is that YALMIP can derive simple optimization models, such as linear programs, instead of treating the operators as general nonlinearities and use a nonlinear solver. We say that the operator is *conic representable* if these reformulations are possible

These graph representations are only possible if the expressions satisfy certain convexity or concavity conditions. Hence, YALMIP has to analyze the problem to ensure that the representation actually can be used.

Graph-based implementations are available for a large number of operators, such as the linear programming representable operators  [min], [max] and [abs], the second-order cone representable  [sqrt], [norm] and [geomean], semidefinite cone representable [nuclear norm], or exponential cone representable such as [entropy]. Adding new operators is easy, and can be done almost entirely from template code.

Read more about [graph models of conic representable function](/tutorial/nonlinearoperatorsgraphs).

### Mixed-integer representations

Mixed-integer representations are models that encode an exact representation of an operator by using integer and binary decision variables. The benefit of using such a representation is that the resulting nonconvex problem typically is a well structured problem, such as a mixed-integer linear program. Many of the conic representable operators that are implemented using linear programming graphs, are also available in a mixed integer representation. If YALMIP fails to propagate convexity, it will switch from graph-based modelling to mixed-integer modelling (unless the option *'allownonconvex'* is set to 0). Mixed-integer models are available for, e.g., [min], [max], [abs], but also for purely combinatorial operators operators and conditions such as [or], [ne], [iff], [implies], [nnz], [sort] and many more, and on a higher level, piecewise affine and quadratic functions in connection with [MPT].

Read more about [integer representable function](/tutorial/nonlinearoperatorsmixedinteger).

### Evaluation-based representations

A third way to model operators in YALMIP is by using simple callback evaluations. This is the way a modelling normally works with nonlinear solver. The modelling layer simply creates a framework for computing function values and derivatives. Operators modelled this way in YALMIP can also be equipped with convexity information, and thus fit into YALMIPs convexity propagations, and for use in the built-in global solver [bmibnb] they can have convex envelope approximators attached.

Since they are based on callbacks, they can only be used together with general purpose nonlinear solvers, such as [FMINCON], [SNOPT] or [IPOPT]. 

Most of MATLABs built-in nonlinear functions, such as [exp] and [sin], are available as evaluation-based representations in YALMIP. There are also operators such as [entropy] and [kullbackleibler] which can be used both with graph representations in [exponentialconeprogramming](/tutorial/exponentialconeprogramming) and as callback operators in general nonlinear solvers.

Read more about [callback operators](/tutorial/nonlinearoperatorsmixedinteger).
