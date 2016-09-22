---
layout: single
category: command
author_profile: false
excerpt: "Built-in meta-solver for some quasi-convex optimization problems"
title: bisection
tags: [Semidefinite programming, Quasi-convex]
comments: true
date: '2016-09-17'
sidebar:
  nav: "commands"
---

[bisection](/command/bisection) implements a bisection algorithm to solve some simple quasi-convex problems.

Assumed constraints quasi-convex in \\(x\\) and the scalar \\(t\\), and objective function \\(t\\) or \\(-t\\)

### Syntax

````matlab
diagnostics = bisection(Constraints,Objective,options)
````

### Example

Bisection can either be called as a command (note that node solver has to be specified)

````matlab
P = sdpvar(2);
sdpvar t
Constraint = [P <= t*P + eye(2), P >= eye(2)/3]
Objective = t;
diagnostics = bisection(Constraint,Objective,sdpsettings('solver','mosek'))
````

or as any other solver

````matlab
P = sdpvar(2);
sdpvar t
Constraint = [P <= t*P + eye(2), P >= eye(2)/3]
Objective = t;
diagnostics = bisection(Constraint,Objective,sdpsettings('solver','bisection','bisection.solver','mosek'))
````

At the moment, YALMIP does not analyze the problem to see if it really is quasi-convex in \\(t\\) and that your specified solver is applicable for fixed \\(t\\). Behaviour when you send an incorrect model is unspecified.

See [the decay-rate example](/example/decayrate) and the [solver specification](/solver/bisection)
