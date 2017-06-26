---
title: "Solvers"
layout: single
permalink: /allsolvers/
sidebar:
  nav: "solvers"
---

One of the core ideas in YALMIP is to rely on external solvers for the low-level numerical solution of optimization problem. YALMIP concentrates on efficient modeling and high-level algorithms.

## Recommended installation

Linear programming can be solved by quadratic programming which can be solved by second-order cone programming which can be solved by semidefinite programming. Hence, in theory, you only need a semidefinite programming solver if you only solve linear problems. In practice though, dedicated solvers are recommended.

A recommended installation if you mainly intend to solve SDPs and LPs and QPs is [MOSEK](/solver/mosek), [SEDUMI](/solver/sedumi) or [SDPT3](/solver/sdpt3).

If you solve non-trivial linear and quadratic programs (and nonconvex problems via [BMIBNB](/solver/bmibnb), a dedicated LP/QP solver is recommended. Most examples in this Wiki have been generated using [MOSEK](/solver/mosek), [GUROBI](/solver/gurobi) and [CPLEX](/solver/cplex). All these solvers have academic licenses giving access to full unlimited versions.

If you intend to solve large problems or other problem classes, you are advised to download several solvers to find one that works best for your problem.

And finally, there are no free lunches and you get what you pay for (unless you're in academia!).

### Available solvers by problem class

A simple categorization is as follows (the definitions of free and commercial depends slightly on the solver, please see the specific comments in the solver description)

### Linear programming (free)
[CDD](solver/cdd), [CLP](/solver/clp), [GLPK](solver/glpk), [LPSOLVE](/solver/lpsolve), [QSOPT](solver/qsopt), [SCIP](/solver/scip)

### Mixed Integer Linear programming (free)
[CBC](solver/cbc), [GLPK](solver/glpk), [LPSOLVE](/solver/lpsolve), [SCIP](/solver/scip)

### Linear programming (commercial)
[CPLEX](/solver/cplex) (free for academia), [GUROBI](/solver/gurobi) (free for academia), [LINPROG](/solver/linprog), [MOSEK](/solver/mosek) (free for academia), [XPRESS](/solver/xpress) (free for academia)

### Mixed Integer Linear programming (commercial)
[CPLEX](/solver/cplex) (free for academia), [GUROBI](/solver/gurobi) (free for academia), [INTLINPROG](/solver/intlinprog), [MOSEK](/solver/mosek) (free for academia), [XPRESS](/solver/xpress) (free for academia)

### Quadratic programming (free)
[OSQP](/solver/osqp), [BPMPD](/solver/bpmpd), [CLP](/solver/clp), [OOQP](/solver/ooqp), [QPC](/solver/qpc), [QPOASES](/solver/qpoases), [QUADPROGBB](solver/quadprogbb) (nonconvex QP)

### Quadratic programming (commercial)
[CPLEX](/solver/cplex) (free for academia), [GUROBI](/solver/gurobi) (free for academia), [MOSEK](/solver/mosek) (free for academia), [NAG](/solver/nag), [QUADPROG](/solver/quadprog), [XPRESS](/solver/xpress) (free for academia)

### Mixed Integer Quadratic programming (commercial)
[CPLEX](/solver/cplex) (free for academia), [GUROBI](/solver/gurobi) (free for academia), [MOSEK](/solver/mosek) (free for academia), [XPRESS](/solver/xpress) (free for academia)

### Second-order cone programming (free)

[ECOS](/solver/ecos), [SDPT3](/solver/sdpt3), [SEDUMI](/solver/sedumi)

## Second-order cone programming (commercial)

[CPLEX](/solver/cplex) (free for academia), [GUROBI](/solver/gurobi) (free for academia), [MOSEK](/solver/mosek) (free for academia)

### Mixed Integer Second-order cone programming (commercial)

[CPLEX](/solver/cplex) (free for academia), [GUROBI](/solver/gurobi) (free for academia), [MOSEK](/solver/mosek) (free for academia)

### Semidefinite programming (free)

[CSDP](/solver/csdp), [DSDP](/solver/dsdp), [LOGDETPPA](/solver/logdetppa), [PENLAB](/solver/penlab), [SDPA](/solver/sdpa), [SDPLR](/solver/sdplr), [SDPT3](/solver/sdpt3), [SDPNAL](/solver/sdpnal), [SEDUMI](/solver/sedumi)

### Semidefinite programming (commercial)

[LMILAB](/solver/lmilab), [MOSEK](/solver/mosek) (free for academia), [PENBMI](/solver/penbmi), [PENSDP](/solver/pensdp) (free for academia)

### General nonlinear programming and other solvers

[BARON](/solver/baron), [FILTERSD](/solver/filtersd), [FMINCON](/solver/fmincon), [GPPOSY](/solver/gpposy), [IPOPT](/solver/ipopt), [KNITRO](/solver/knitro), [LMIRANK](/solver/lmirank), [MPT](/solver/mpt), [NOMAD](/solver/nomad), [PENLAB](/solver/penlab), [SNOPT](/solver/snopt), [SPARSEPOP](/solver/sparsepop)

## Internal solvers

By exploiting the optimization infrastructure in YALMIP, it is fairly easy to develop algorithms based on the external solvers. This has motivated development of mixed integer conic solvers ([BNB](/solver/bnb), [CUTSDP](/solver/cutsdp)), indefinite quadratic programming ([BMIBNB](/solver/bmibnb), [KKTQP](/solver/kktqp) ), simple quasi-convex problems ([bisection](/command/bisection)), sum-of-squares and semidefinite relaxation modules ([solvesos](/command/solvesos), [solvemoment](/command/solvemoment))  and a global solver for pretty much any nonlinear problem ([BMIBNB](/solver/bmibnb)), just to mention some of the most important modules in YALMIP.
