---
layout: single
permalink: /Extracting
excerpt: "Extracting inputs and outputs from solvers"
title: "Extracting low-level data from solvers"
tags: [Export and import, Low-level]
comments: true
date: '2017-08-11'
---

In normal cases, any communication between YALMIP and the solver is uninteresting, as all we are interested in is the [diagnostic code](command/yalmiperror) available in the output from [optimize](/command/optimize) and optimal values which are asigned to our decision variables and available through [value](/command/value). There are however cases where we want to obtain all the information the solver has returned in the call, and the model we sent to the solver.

# Extracting solver input

If we want to investigate the model that YALMIP sends to a solver, we can use either the command [export](/command/export), or call the solver and use the options 'savesolveroutput' or 'debug'.

To extract a numerical model is solver-specific format without calling the solver, we use [export](/command/export)
````matlab
sdpvar x y
Model = export([x >= 0, x+y == 1], x^2 + y,sdpsettings('solver','quadprog'))

Model = 

      Q: [2x2 double]
      c: [2x1 double]
      A: [-1 0]
      b: 0
    Aeq: [1 1]
    beq: 1
     lb: [2x1 double]
     ub: [2x1 double]
    ops: [1x1 struct]
     x0: []
      f: 0
````

Alternatively, we might be interested in the model that was sent to the solver, when we actually call it.
````matlab
sdpvar x y
diagnostics = optimize([x >= 0, x+y == 1], x^2 + y,sdpsettings('solver','quadprog','savesolverinput',1));
diagnostics.solverinput

ans = 

      Q: [2x2 double]
      c: [2x1 double]
      A: [-1 0]
      b: 0
    Aeq: [1 1]
    beq: 1
     lb: [2x1 double]
     ub: [2x1 double]
    ops: [1x1 struct]
     x0: []
      f: 0
````

This model can also be obtained by using the 'savedebug' option which will create a file with the model, before calling the solver.

````matlab
sdpvar x y
diagnostics = optimize([x >= 0, x+y == 1], x^2 + y,sdpsettings('solver','quadprog','savedebug',1));
load debugfile
model

model = 

      Q: [2x2 double]
      c: [2x1 double]
      A: [-1 0]
      b: 0
    Aeq: [1 1]
    beq: 1
     lb: [2x1 double]
     ub: [2x1 double]
    ops: [1x1 struct]
     x0: []
      f: 0
````


# Extracting solver output

In some cases, we might see some information displayed by the solver, such as gap metrics, feasibility distances, iteration counters, and we want to obtain those numbers. The only way to do this, beyond some clever regexp magic on the text in the command window, is to use the option 'savesolveroutput'. When doing so, all information that the solver returns in MATLAB, will be available in the output structure returned in [optimize](/command/optimize) 

````matlab
sdpvar x y
diagnostics = optimize([x >= 0, x+y == 1], x^2 + y,sdpsettings('solver','quadprog','savesolverinput',1));
diagnostics.solveroutput

ans = 

         x: [2x1 double]
      fmin: 0.7500
      flag: 1
    output: [1x1 struct]
    lambda: [1x1 struct]

````

If the information of interest isn't available here, it is not possible to extract it through the MATLAB interface.
