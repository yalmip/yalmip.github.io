---
title: "Solvers"
layout: single
---

One of the core ideas in YALMIP is to rely on external solvers for the low-level numerical solution of optimization problem. YALMIP concentrates on efficient modeling and high-level algorithms.

## Recommended installation

Linear programming can be solved by quadratic programming which can be solved by second-order cone programming which can be solved by semidefinite programming. Hence, in theory, you only need a semidefinite programming solver if you only solve linear problems.

A recommended installation if you mainly intend to solve SDPs and perhaps some small to medium sized LPs and QPs is [SEDUMI](sedumi/) and [SDPT3](sdpt3.html) (these solvers are by no means the best solvers, but they support second order cones which makes them fairly efficient also for QPs).

If you solve non-trivial linear and quadratic programs (and nonconvex problems via [[Solvers.BMIBNB | BMIBNB]]), a dedicated LP/QP solver is recommended. Most examples in this Wiki have been generated using [[GUROBI]].

If you are in academia, [[MOSEK]] is convenient since it gives you a very competetive MILP, MIQP, MISOCP and SDP solver in one package. 
If you intend to solve large problems or other problem classes, you are advised to download several solvers to find one that works best for your problem class.

And finally, there are no free lunches and you get what you pay for, most often...

A simple categorization is as follows (the definitions of free and commercial depends slightly on the solver, please see the specific comments in the solver description)

## Linear programming (free)
[[CDD]], [[CLP]], [[GLPK]], [[LPSOLVE]], [[QSOPT]], [[SCIP]]

## Mixed Integer Linear programming (free)
[[CBC]], [[GLPK]], [[LPSOLVE]], [[SCIP]]

## Linear programming (commercial)
[[CPLEX]] (free for academia), [[GUROBI]] (free for academia), [[LINPROG]], [[MOSEK]] (free for academia), [[XPRESS]] (free for academia)

## Mixed Integer Linear programming (commercial)
[[CPLEX]] (free for academia), [[GUROBI]] (free for academia), [[MOSEK]] (free for academia), [[XPRESS]] (free for academia)

## Quadratic programming (free)
[[BPMPD]], [[CLP]], [[OOQP]], [[QPC]], [[qpOASES]], [[quadprogBB]] (nonconvex QP)

## Quadratic programming (commercial)
[[CPLEX]] (free for academia), [[GUROBI]] (free for academia), [[MOSEK]] (free for academia), [[NAG]], [[QUADPROG]], [[XPRESS]] (free for academia)

## Mixed Integer Quadratic programming (commercial)
[[CPLEX]] (free for academia), [[GUROBI]] (free for academia), [[MOSEK]] (free for academia), [[XPRESS]] (free for academia)

## Second-order cone programming (free)

[[ECOS]], [[SDPT3]], [[SEDUMI]]

## Second-order cone programming (commercial)

[[CPLEX]] (free for academia), [[GUROBI]] (free for academia), [[MOSEK]] (free for academia)

## Mixed Integer Second-order cone programming (commercial)

[[CPLEX]] (free for academia), [[GUROBI]] (free for academia), [[MOSEK]] (free for academia)

## Semidefinite programming (free)

[[CSDP]], [[DSDP]], [[LOGDETPPA]], [[PENLAB]], [[SDPA]], [[SDPLR]], [[SDPT3]], [[SDPNAL]], [[SEDUMI]]

## Semidefinite programming (commercial)

[[LMILAB]], [[MOSEK]] (free for academia), [[PENBMI]], [[PENSDP]] (free for academia)

## General nonlinear programming and other solvers

[[BARON]], [[FILTERSD]], [[FMINCON]], [[GPPOSY]], [[IPOPT]], [[KNITRO]], [[KYPD]], [[LMIRANK]], [[MPT]], [[NOMAD]], [[PENLAB]], [[SNOPT]], [[STRUL]], [[VSDP]], [[SparsePOP]]

## Internal solvers

By exploiting the optimization infrastructure in YALMIP, it is fairly easy to develop algorithms based on the external solvers. This has motivated development of mixed integer conic solvers ([[Solvers.BNB |bnb]], [[CUTSDP | cutsdp]]), global solvers for general nonlinear problems ([[Solvers.BMIBNB| bmibnb]]), indefinite quadratic programming ([[Solvers.BMIBNB| bmibnb]], [[Solvers.KKTQP| kktqp]] ), sum-of-squares modules ([[Tutorials.SumOfSquares | solvesos]]), just to mention some of the most important contributions in YALMIP.
