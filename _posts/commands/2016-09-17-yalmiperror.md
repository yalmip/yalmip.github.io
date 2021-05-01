---
category: command
excerpt: ""
title: yalmiperror
tags:
date: '2016-09-17'
sidebar:
  nav: "commands"
---

[yalmiperror](/command/yalmiperror) returns a description of the error codes used in [optimize](/command/optimize) etc.

### Syntax

````matlab
string = yalmiperror(n)
````

### Examples
To se all available error codes, simply issue the command without any input.
````matlabb
yalmiperror

  Error codes
  -9 Specified solver name not recognized
  -8 Problem does not satisfy geometric programming rules
  -7 Solver does not return error codes
  -6 Search space not bounded (bound all variables)
  -5 License problems in solver
  -4 Solver not applicable
  -3 Solver not found
  -2 No suitable solver
  -1 Unknown error
   0 Successfully solved
   1 Infeasible problem
   2 Unbounded objective function
   3 Maximum iterations exceeded
   4 Numerical problems
   5 Lack of progress
   6 Initial solution infeasible
   7 YALMIP sent incorrect input to solver
   8 Feasibility cannot be determined
   9 Unknown problem in solver
  10 bigM failed (increase sp.Mfactor)  
  11 Other identified error
  12 Infeasible or unbounded
  13 YALMIP cannot determine status in solver
  14 Model creation failed
  15 Problem either infeasible or unbounded
  16 User terminated
  17 Presolve recovery failed
````

To retrieve a description of a particular error code
````matlabb
yalmiperror(3)
 ans =
 Maximum iterations exceeded
````
