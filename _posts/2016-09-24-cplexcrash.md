---
layout: single
permalink: /R20160923
excerpt: "It's been a while..."
title: "MATLAB 2016 + CPLEX crash"
tags: [Release]
comments: true
date: '2016-09-23'
---

CPLEX and/or MATLAB 2016 has a bug which causes this combination to cause a seg-fault when YALMIP is used.

The culprit is the function [sdpsettings](/command/sdpsettings) which crashes violently, and this function is called when YALMIP sets up an options structure, no matter solver you will use. The problem is the function **cplexoptimset**.

Possible work-arounds

* Edit **sdpsettings.m** and change **cplex = cplexoptimset('cplex');** to **cplex = cplexoptimset;**. The drawback is that the options structure reduced, i.e., this call does not create a cmplete set of options

* Alternatively, look around for another solver. [GUROBI] and [MOSEK] has a much better history of fixing their critical bugs rapidly.
