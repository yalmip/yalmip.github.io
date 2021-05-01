---
category: command
excerpt: "Options structure creation"
title: sdpsettings
tags: Options
date: '2016-09-17'
sidebar:
  nav: "commands"
---

[sdpsettings](/command/sdpsettings) is used to communicate options to YALMIP and solvers. It is used as the third argument in commands such [optimize](/command/optimize), [optimizer](/command/optimizer), [solvesos](/command/solvesos), [solvemoment](/command/solvemoment) and [solvemp](/command/solvemp). 

### Syntax


````matlab
options = sdpsettings('field',value,'field',value,...)
optimize(Constraints, Objective, options)
```` 

### Examples

Creating an options structure with specified values is easily done.

````matlab
ops = sdpsettings('solver','mosek','verbose',1,'debug',1)
```` 

A convenient way to alter many options without getting a long line is to send an existing options structure as the first input argument.

````matlab
ops = sdpsettings('solver','sdpa');
ops = sdpsettings(ops,'verbose',0);
```` 

Alternatively, the options structure can be manipulated directly

````matlab
ops = sdpsettings('solver','sdpa');
ops.verbose = 0;
```` 

Solver-specific options

````matlab
ops = sdpsettings('solver','sdpa','sdpa.maxIteration',100);
```` 

The easiest way to find out the possible options is to define a default options structure and display it

````matlab
ops = sdpsettings;

ops
               solver: ''
              verbose: 1
              warning: 1
         cachesolvers: 0
                debug: 1
        beeponproblem: [-5 -4 -3 -2 -1]
         showprogress: 0
            saveduals: 1
     removeequalities: 0
     savesolveroutput: 0
      savesolverinput: 0
    convertconvexquad: 1
               radius: Inf
                relax: 0
                usex0: 0             
            savedebug: 0
                  sos: [1x1 struct]
               moment: [1x1 struct]
                  bnb: [1x1 struct]
                bpmpd: [1x1 struct]
               bmibnb: [1x1 struct]
               cutsdp: [1x1 struct]
               global: [1x1 struct]
                  cdd: [1x1 struct]
                  clp: [1x1 struct]
                cplex: [1x1 struct]
                 csdp: [1x1 struct]
                 dsdp: [1x1 struct]
                 glpk: [1x1 struct]
                 kypd: [1x1 struct]
               lmilab: [1x1 struct]
              lmirank: [1x1 struct]
              lpsolve: [1x1 struct]
               maxdet: [1x1 struct]
                  nag: [1x1 struct]
               penbmi: [1x1 struct]
               pennlp: [1x1 struct]
               pensdp: [1x1 struct]
                 sdpa: [1x1 struct]
                sdplr: [1x1 struct]
                sdpt3: [1x1 struct]
               sedumi: [1x1 struct]
                qsopt: [1x1 struct]
               xpress: [1x1 struct]
             quadprog: [1x1 struct]
              linprog: [1x1 struct]
             bintprog: [1x1 struct]
              fmincon: [1x1 struct]
           fminsearch: [1x1 struct]

ops.sedumi

ans = 

          alg: 2
         beta: 0.5000
        theta: 0.2500
         free: 1
          sdp: 0
      stepdif: 0
            w: [1 1]
           mu: 1
          eps: 1.0000e-09
       bigeps: 1.0000e-03
      maxiter: 150
        vplot: 0
       stopat: -1
         denq: 0.7500
         denf: 10
       numtol: 5.0000e-07
    bignumtol: 0.9000
      numlvlv: 0
         chol: [1x1 struct]
           cg: [1x1 struct]
    maxradius: Inf
```` 

### solver

In the code above, we told YALMIP to use the solver Mosek. The possible values to give to the field **solver** can be found in the [solver documentation](/allsolvers). If the solver isn't found, an [error code](/command/yalmiperror) will be returned in the output structure. To let YALMIP select the solver, use the default solver tag ''. If you give a comma-separated list of solvers such as **'dsdp,csdp,sdpa'**, YALMIP will select based on this preference. If you add a wildcard in the end **'dsdp,csdp,sdpa,*'**, YALMIP will select another solver if none of the solvers in the list were found. 

### verbose

By setting **verbose** to 0, the solvers will run with minimal display. By increasing the value, the display level is controlled (typically 1 gives modest display level while 2 gives an awful amount of information printed).

### debug

If **debug** is turned on, YALMIP will not try to catch errors, which will simplify finding out where and why YALMIP failed  unexpectedly.

### warning

The **warning** option can be used to get a warning displayed when a solver has run into some kind of problem (recommended to be used if the solver is run in silent mode).  

### beeponproblem

The field **beeponproblem** contains a list of error codes (see [yalmiperror](/command/yalmiperror)). YALMIP will beep if any of these errors occurs (nice feature if you're taking a coffee break during heavy calculations).

### showprogress

When the field **showprogress** is set to 1, the user can see what YALMIP currently is doing (might be a good idea for large-scale problems). 

### cachesolvers

Every time [optimize](/command/optimize) is called, YALMIP checks for available solvers. This can take a while on some systems (some slow networks), so it is possible to avoid doing this check every call. Set **cachesolvers** to 1, and YALMIP will remember the solvers (using a persistent variable) found in the first call to [optimize](/command/optimize). If solvers are added to the path after the first call, YALMIP will not detect this. Hence, after adding a solver to the path the work-space must be cleared or [optimize](/command/optimize) must be called once with cachesolvers set to 0. Only use this option if you absolutely have to, this is practically obsolete on modern systems.

### removeequalities

When the field **removeequalities** is set to 1, YALMIP removes equality constraints using a QR decomposition and reformulates the problem using a smaller set of variables.

If **removeequalities** is set to 2, YALMIP removes equality constraints using a basis derived directly from independent columns of the equality constraints (higher possibility of maintaining sparsity than the QR approach, but may lead to a numerically poor basis). 

With **removeequalities** set to -1, equalities are removed by YALMIP by converting them to double-sided inequalities. When set to 0 (default), YALMIP does nothing if the solver supports equalities. If the solver does not support equalities, YALMIP uses double-sided inequalities.

### saveduals

If **saveduals** is set to 0, the dual variables will not be saved in YALMIP. This might be useful for large sparse problems with a dense dual variable. Setting the field to 0 will then save some memory.

### savesolverinput, savesolveroutput

The fields **savesolverinput** and **savesolveroutput** can be used to see what is actually sent to and returned from the solver. This data will then be available in the output structure from [optimize](/command/optimize).

### convertconvexquad

With **convertconvexquad** set to 1 (default), YALMIP will try to convert quadratic constraints to second order cones.

### relax

If **relax** is set to 1, all nonlinearities and integrality constraints will be disregarded. Integer variables are relaxed to continuous variables and nonlinear variables are treated as independent variables (i.e., **x** and **x^2** will be treated as two separate variables). If set to 2, only integrality constraints are relaxed, while set to 3 only nonlinearities are relaxed.

### usex0

The current solution (the value returned from the [value](/command/value) command) can be used as an initial guess when solving an optimization problem. Setting the field **usex0** to 1 tells YALMIP to supply the current values as an initial guess to the solver. You can manually specify a current value using [assign](/command/assign).

### solver options

The options structure also contains a number of structures with parameters used in the specific solver. As an example, the following parameters can be tuned for [SEDUMI](/solver/sedumi) (for details, the user is referred to the manual of the specific solver).

````matlab
ops.sedumi
ans = 
                 alg: 2
               theta: 0.2500
                beta: 0.5000
                 eps: 1.0000e-009
              bigeps: 0.0010
              numtol: 1.0000e-005
                denq: 0.7500
                denf: 10
               vplot: 0
             maxiter: 100
             stepdif: 1
                   w: [1 1]
              stopat: -1
                  cg: [1x1 struct]
                chol: [1x1 struct]
````

Hence, the following code will select SeDuMi and change the default precision

````matlab
ops = sdpsettings('solver','sedumi','sedumi.eps',1e-12);
````
