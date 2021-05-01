---
title: "BISECTION"
category: solver
layout: single-solver
tags: [Semidefinite programming solver, Quasi-convex]
excerpt: "Built-in solver for simple quasi-convex programs"
developer: "J. Löfberg"
sidebar:
  nav: "solvers"
---

The solver relies on external solvers for solving the bounding problems during the bisection phase.

Note that the solver assumes you know what you are doing and actually have a problem which is quasi-convex in some variables \\(x\\) and a scalar \\(t\\), and that your objective is \\(t\\) or \\(-t\\)
