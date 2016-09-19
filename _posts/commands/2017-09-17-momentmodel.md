---
layout: single
category: command
author_profile: false
excerpt: "Derive semidefinite relaxation model of polynomial program without solving it"
title: momentmodel
tags: [Moment relaxations, Semidefinite relaxations]
comments: true
date: '2016-09-17'
sidebar:
  nav: "commands"
---

[momentmodel] derives the semidefinite relaxation model of a polynomial program without solving it.

### Syntax

````matlab
[ConRelaxed, ObjRelaxed] = momentmodel(Con,Obj,RelaxationOrder)
````

### Example

Consider the following polynomial program

````matlab
sdpvar x y
Con = [x^4 + y^4 + x*y <= 1];
Obj = x^2 +y^3;
````

We can compute lower bounds using [solvemoment], or alternatively derive the semidefinite relaxation (here lowest order possible) and proceed manually.

````matlab
[ConRelaxed, ObjRelaxed,MomentMatrices] = momentmodel(Con,Obj)
optimize(ConRelaxed,ObjRelaxed);
value(MomentMatrices{3})
````





