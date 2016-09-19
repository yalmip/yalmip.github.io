---
title: "Exponential cone programming"
category: tutorial
author_profile: false
level: 4
tags: [Exponential cone programming]
excerpt: "Convex conic optimization over exponentials and logarithms"
layout: single
sidebar:
  nav: "tutorials"
---

The exponential cone is defined as the set \\(  (ye^{x/y}\leq z, y>0) \\). YALMIP is capable of detecting and calling specialized solvers for a variety of exponential cone representable function. 

The following example requires [SCS](/solver/scs) or [ECOS](/solver/ecos) to be relavant. If none of those solvers are installed, YALMIP will work with the nonlinear functions as written and treat the problem as a general nonlinear program.

By simple variable transformations, the following functions are automatically detected as exponential cone representable and suitably rewritten before calling an exponential cone capable solver

1. [exp], [pexp]
2. [log], [log2], log10], [slog], [plog]
3. [entropy], [logsumeexp], [kullbackleibler]

Note that YALMIP does not detect exponential cones when written in the canonical form \\( ye^{x/y}\leq z \\).
````matlab
````

If the exponential cone program violates convexity rules, and an exponential cone solver is selected an error will be issued

````matlab
sdpvar x
optimize([],log(x), sdpsettings('solver',scs')
````
