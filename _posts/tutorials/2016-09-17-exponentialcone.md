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

The exponential cone is defined as the set \\(  (ye^{x/y}\leq z, y>0) \\), see, e.g. [Chandrasekara and Shah 2015]. YALMIP is capable of detecting and calling specialized solvers for a variety of exponential cone representable function. 

By simple variable transformations, the following functions are automatically detected as exponential cone representable and suitably rewritten before calling an exponential cone capable solver

1. [exp](/command/exp), [pexp](/command/exp)
2. [log](/command/log), [log2](/command/log), log10](/command/log), [slog](/command/log), [plog](/command/log)
3. [entropy](/command/entropy), [logsumexp](/command/logsumexp), [kullbackleibler](/command/kullbackleibler)

Note that YALMIP does not detect exponential cones when written in the canonical form \\( ye^{x/y}\leq z \\), but instead you can use the perspective exponential, [pexp], which implements  \\( x_1e^{x_1/x_2} \\).

The code below requires [SCS](/solver/scs) or [ECOS](/solver/ecos) to be relavant. If none of those solvers are installed, YALMIP will work with the nonlinear functions as written and treat the problem as a general nonlinear program.

````matlab
````

If the exponential cone program violates convexity rules, and an exponential cone solver is selected an error will be issued

````matlab
sdpvar x
optimize([],log(x), sdpsettings('solver',scs')
````
