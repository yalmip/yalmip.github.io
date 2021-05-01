---
category: faq
title: I have problems solving a geometric program
tags: [Geometric programming]
date: '2016-09-17'
sidebar:
  nav:
---

To begin with, you need a solver that can solve geometric programs (YALMIP currently supports [GPPOSY](/solver/gpposy) and [MOSEK](/solver/mosek), [FMINCON](/solver/fmincon), [IPOPT](/solver/ipopt), and [SNOPT](/solver/snopt) by internally performing a variable change to write the problem as a convex problem over logarithmic variables). If you still have problems, the reason may be that YALMIP converts convex quadratic constraints to seconds order constraints. This should not be done in geometric programs (it is a known bug). To avoid this, set the option **'convertconvexquad'** to 0. Another reason is that runs into problem during the expansion of nonlinear operators. To avoid this issue, explicitly tell YALMIP that the problem is a geometric problem by specifying the solver to **'gpposy'**, **'mosek-geometric'**, **'fmincon-geometric'**, **'snopt-geometric'** or **'ipopt-geometric'**.
