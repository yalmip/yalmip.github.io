---
layout: single
permalink: /R20160923
excerpt: "Boom!"
title: "MATLAB 2016 + CPLEX crash"
tags: 
comments: true
date: '2016-09-23'
---

[CPLEX](/solver/cplex) and/or MATLAB 2016 has a bug which causes this combination to cause a seg-fault when YALMIP is used.

The crash apears inside [sdpsettings](/command/sdpsettings) which crashes violently, and this function is called when YALMIP sets up an options structure, no matter solver you will use. The problem is the function **cplexoptimset** supplied by [CPLEX](/solver/cplex)

Possible work-arounds

* Edit **sdpsettings.m** and change **cplex = cplexoptimset('cplex');** to **cplex = cplexoptimset;**. The drawback is that the options structure reduced, i.e., this call does not create a cmplete set of options

* Alternatively, look around for another solver. [GUROBI](/solver/gurobi) and [MOSEK](/solver/mosek) has a much better history of fixing their critical bugs rapidly.

An upcoming patch-release of YALMIP will try to code arond these issues.
