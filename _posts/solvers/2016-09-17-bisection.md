---
title: "BISECTION"
category: solver
layout: single
author_profile: false
tags: [Semidefinite programming solver, Quasi-convex]
sidebar:
  nav: "solvers"
---

Built-in meta-solver for some simple quasi-convex problems.

### YALMIP

BISECTION is invoked by running the command [bisection](/commands/bisection)

Future version might support  `sdpsettings('solver','bisection')`

### Comments

The solver relies on external solvers for solving the bounding problems during the bisection phase.

Note that the solver assumes you know what you are doing and actually have a problem which is quasi-convex in some variables \\(x\\) and a scalar \\(t\\), and that your objective is \\(t\\) or \\(-t\\)
