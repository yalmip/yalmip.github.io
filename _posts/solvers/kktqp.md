---
title: "KKTQP"
layout: single
permalink: /solvers/kktqp/
sidebar:
  nav: "solvers"
---

Built-in global solver for nonconvex quadratic programming problems.

### YALMIP
KKTQP is invoked with `sdpsettings('solver','kktqp')`

### Comments
KKTQP is an experimental implementation with little testing performed. It is based on solving the mixed integer linear program arising from the KKT conditions of the quadratic problem, using fast external mixed integer linear programming solvers.

See [nonconvex QP discussion and comparison](/Nonconvex-quadratic-programming)
