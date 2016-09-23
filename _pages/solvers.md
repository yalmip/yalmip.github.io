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

A recommended installation if you mainly intend to solve SDPs and LPs and QPs is [MOSEK](/solver/mosek), [SEDUMI] or [SDPT3].

If you solve non-trivial linear and quadratic programs (and nonconvex problems via [BMIBNB](/solver/bmibnb), a dedicated LP/QP solver is recommended. Most examples in this Wiki have been generated using [MOSEK](/solver/mosek), [GUROBI](/solver/gurobi) and [CPLEX].

If you are in academia, [MOSEK](/solver/mosek) is convenient since it gives you a very competetive MILP, MIQP, MISOCP and SDP solver in one package.

If you intend to solve large problems or other problem classes, you are advised to download several solvers to find one that works best for your problem.

And finally, there are no free lunches and you get what you pay for (unless you're in academia!).

### Available solvers by problem class

A simple categorization is as follows (the definitions of free and commercial depends slightly on the solver, please see the specific comments in the solver description)

### Linear programming (free)
[CDD](solver/cdd), [CLP](/solver/clp), [GLPK](solver/glpk), [LPSOLVE](/solver/lpsolve), [QSOPT](solver/qsopt), [SCIP](/solver/scip)

### Mixed Integer Linear programming (free)
[CBC](solver/cbc), [GLPK](solver/glpk), [LPSOLVE](/solver/lpsolve), [SCIP](/solver/scip)

### Linear programming (commercial)
[CPLEX](/solver/cplex) (free for academia), [GUROBI](/solver/gurobi) (free for academia), [LINPROG], [MOSEK](/solver/mosek) (free for academia), [XPRESS](/solver/xpress) (free for academia)

### Mixed Integer Linear programming (commercial)
[CPLEX](/solver/cplex) (free for academia), [GUROBI](/solver/gurobi) (free for academia), [MOSEK](/solver/mosek) (free for academia), [XPRESS](/solver/xpress) (free for academia)

### Quadratic programming (free)
[BPMPD](/solver/bpmpd), [CLP](/solver/clp), [OOQP](/solver/ooqp), [QPC](/solver/qpc), [QPOASES](/solver/qpoases), [QUADPROGBB](solver/quadprogbb) (nonconvex QP)

### Quadratic programming (commercial)
[CPLEX](/solver/cplex) (free for academia), [GUROBI](/solver/gurobi) (free for academia), [MOSEK](/solver/mosek) (free for academia), [NAG](/solver/nag), [QUADPROG](/solver/quadprog), [XPRESS](/solver/xpress) (free for academia)

### Mixed Integer Quadratic programming (commercial)
[CPLEX](/solver/cplex) (free for academia), [GUROBI](/solver/gurobi) (free for academia), [MOSEK](/solver/mosek) (free for academia), [XPRESS](/solver/xpress) (free for academia)

### Second-order cone programming (free)

[ECOS](/solver/ecos), [SDPT3], [SEDUMI]

## Second-order cone programming (commercial)

[CPLEX](/solver/cplex) (free for academia), [GUROBI](/solver/gurobi) (free for academia), [MOSEK](/solver/mosek) (free for academia)

### Mixed Integer Second-order cone programming (commercial)

[CPLEX](/solver/cplex) (free for academia), [GUROBI](/solver/gurobi) (free for academia), [MOSEK](/solver/mosek) (free for academia)

### Semidefinite programming (free)

[CSDP](/solver/csdp), [DSDP](/solver/dsdp), [LOGDETPPA](/solver/logdetppa), [PENLAB](/solver/penlab), [SDPA](/solver/sdpa), [SDPLR](/solver/sdplr), [SDPT3], [SDPNAL], [SEDUMI]

### Semidefinite programming (commercial)

[LMILAB], [MOSEK](/solver/mosek) (free for academia), [PENBMI](/solver/penbmi), [PENSDP] (free for academia)

### General nonlinear programming and other solvers

[BARON](/solver/baron), [FILTERSD](/solver/filtersd), [FMINCON](/solver/fmincon), [GPPOSY](/solver/gpposy), [IPOPT](/solver/ipopt), [KNITRO](/solver/knitro), [KYPD], [LMIRANK](/solver/lmirank), [MPT](/solver/mpt), [NOMAD], [PENLAB](/solver/penlab), [SNOPT](/solver/snopt), [STRUL], [VSDP], [SPARSEPOP](/solver/sparsepop)

## Internal solvers

By exploiting the optimization infrastructure in YALMIP, it is fairly easy to develop algorithms based on the external solvers. This has motivated development of mixed integer conic solvers ([BNB], [CUTSDP]), global solvers for general nonlinear problems ([BMIBNB]), indefinite quadratic programming ([BMIBNB](/solver/bmibnb), [KKTQP](/solver/kktqp) ), simple quasi-convex problems ([bisection]), sum-of-squares modules ([solvesos]), just to mention some of the most important modules in YALMIP.
