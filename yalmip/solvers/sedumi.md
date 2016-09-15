---
title: "SeDuMi"
layout: single
sidebar:
  nav: "solvers"
---

SeDuMi is freely distributed and available with source and binaries from [github](https://github.com/SQLP/SeDuMi). The official site is <http://sedumi.ie.lehigh.edu> but it lacks some binaries and does not necessarily include the latest bug fixes.

### YALMIP
SeDuMi is invoked with @@sdpsettings('solver','sedumi')@@.

### Solver & algorithm documentation
[USING SEDUMI 1.02, A MATLAB TOOLBOX FOR OPTIMIZATION OVER SYMMETRIC CONES](http://www.optimization-online.org/DB_HTML/2001/10/395.html)

[Implementation of Interior Point Methods for Mixed Semidefinite and Second Order Cone Optimization Problems](http://www.optimization-online.org/DB_HTML/2002/08/518.html)

### Comments
SeDuMi is the most commonly used solver among YALMIP users.

Please note that there are issues with recent versions of MATLAB and 64-bit SeDuMi. Before you use SeDuMi with YALMIP, make sure that your SeDuMi installation actually works, by running some of the examples included in the SeDuMi distribution.
