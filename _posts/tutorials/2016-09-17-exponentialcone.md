---
title: "Exponential cone programming"
category: tutorial
author_profile: false
level: 4
tags: [Exponential cone programming, Relative entropy programming, Exponential and logarithmic functions, Logistic regression]
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

### Logistic regression example

As an example, we solve the logistic regression problem. The problem here is to find a classifier \\( \operatorname{sign}(a^Tx + b)\\) for a given dataset \\( x_i\\) with associated labels \\( y_i = \pm 1 \\). In other words, a classifier similiar to the simple separating hyperplane discussed in the [linear programming tutorial](/tutorials/linearprogramming). 

A convex relaxation of the problem can be solved by minimizing \\( \sum \log(1 + e^{-y_i(a^Tx_i + b)}) \\). This can be written as a sum of [logsumexp](/command/logsumexp) operators by noting that \\( \log(1 + e^{z})=\log(e^{0} + e^{z}) \\). In YALMIP, you do not have to think further, but can use the [logsumexp](/command/logsumexp) operator directly to solve the problem. However, let us show how this boils down to a simple exponential cone program.

To begin with, the problem can be written as minimizing \\( \sum t_i \\) subject to \\( \log (e^0 + e^{-y_i(a^Tx_i + b)}) \leq t_i \\). The constraint is first rewritten by exponentiating both sides to  \\(e^0 + e^{-y_i(a^Tx_i + b)} \leq e^{t_i} \\), which is equivalent to  \\(e^{-t_i} + e^{-y_i(a^Tx_i + b)-t_i} \leq 1 \\), which can be further normalized to \\(z_i + u_i \leq 1, e^{-t_i}\leq z_i, e^{-y_i(a^Tx_i + b)-t_i} \leq u_i\\), which thus lands us in an exponential cone program.

Generate data as in the [linear programming tutorial](/tutorials/linearprogramming) for a classification problem. 

````matlab
%
````

Solve the problem using the built-in  [logsumexp](/command/logsumexp) operator which automatically models the problem as an exponential cone program if an exponential cone program solver is specified. Note that [logsumexp](/command/logsumexp) applied to a matrix will return a vector with  [logsumexp](/command/logsumexp) applied to every row. 

````matlab
%
````

Alternatively, set up the low-level model manually




### Comments

If the exponential cone program violates convexity rules, and an exponential cone solver is selected an error will be issued

````matlab
sdpvar x
optimize([],log(x), sdpsettings('solver',scs'))
````
