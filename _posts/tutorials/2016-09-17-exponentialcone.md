---
title: "Exponential cone programming"
category: tutorial
author_profile: false
level: 4
tags: [Exponential cone programming, Exponential and logarithmic functions]
excerpt: "Convex conic optimization over exponentials and logarithms"
layout: single
sidebar:
  nav: "tutorials"
---

The exponential cone is defined as the set \\(  (ye^{x/y}\leq z, y>0) \\), see, e.g. [Chandrasekara and Shah 2016](/reference/chandrasekaran2016) for a primer on exponential cone programming and the equivalent framework of relative entropy programming. YALMIP is capable of detecting and calling specialized solvers for a variety of exponential cone representable function. 

By simple variable transformations, the following functions are automatically detected as exponential cone representable and suitably rewritten before calling an exponential cone capable solver

1. [exp](/command/exp), [pexp](/command/pexp)
2. [log](/command/log), [log2](/command/log), [log10](/command/log), [slog](/command/log), [plog](/command/plog)
3. [entropy](/command/entropy), [logsumexp](/command/logsumexp), [kullbackleibler](/command/kullbackleibler)

Note that YALMIP does not necessarily detect exponential cones when written in the canonical form \\( ye^{x/y}\leq z \\), but instead you can use the perspective exponential, [pexp](/command/pexp), which implements  \\( ye^{x/y} \\).

The code below requires [SCS](/solver/scs) or [ECOS](/solver/ecos) to be relevant. If none of those solvers are installed, YALMIP will work with the nonlinear functions as written and treat the problem as a general nonlinear program.


### Logistic regression

As an example, we solve the logistic regression problem. The problem here is to find a classifier $\\( \sgn (a^Tx + b)\\) for a given dataset \\( x_i\\) with labels \\( y_i \in \{-1,1\} \\). In other words, a harder classifier than the simple separating hyperplance discussed in the [linear programming tutorial](). 

A convex relaxation of the problem can be solved by minimizing \\( \sum \log(1 + e^{-y_i(a^Tx_i + b}) \\). We note that this can be written as sum of [logsumexp](/command/logsumexp) operators by noting that \\( \log(1 + e^{z}) \\) also can be written as  \\( \log(e^{0} + e^{z}) \\). In YALMIP, you do not have to think further, but use the [logsumexp](/command/logsumexp) operator directly. However, let us show how this boils down to a simple exponential cone program.

To begin with, the problem can be written as minimizing \\( \sum t_i \\) subject to \\( \log (e^0 + e^{-y_i(a^Tx_i + b}) \leq t_i \\). The constraint is rewritten by exponentiating both sides to  \\(e^0 + e^{-y_i(a^Tx_i + b}) \leq e^t_i \\), which is equivalent to  \\(e^{-t_i} + e^{-y_i(a^Tx_i + b}-t_i) \leq 1 \\), which can be further normalized to \\(z_i + u_i \leq 1, e^{-t_i}\leq z_i, e^{-y_i(a^Tx_i + b}-t_i) \leq u_i\\), which thus lands us in an exponential cone program.


````matlab
%
````


### Comments

If the exponential cone program violates convexity rules, and an exponential cone solver is selected an error will be issued

````matlab
sdpvar x
optimize([],log(x), sdpsettings('solver',scs'))
````
