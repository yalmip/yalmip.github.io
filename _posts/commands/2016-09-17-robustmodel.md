---
category: command
excerpt: "Derive robust counterpart model of uncertain program"
title: robustmodel
tags: [Robust optimization]
date: '2016-09-17'
sidebar:
  nav: "commands"
---

[robustmodel](/command/robustmodel) is used to derive a robust counterpart of an uncertain optimization problem without solving it.

### Syntax

````matlab
 [Frobust,robustobjective] = robustify(F,objective,options)
````

### Examples

Consider the following uncertain problem (**w** is the uncertain variable)

````matlab
sdpvar x w
F = [x+w <= 1];
W = [uncertain(w),-0.5 <= w <= 0.5];
objective = -x;
````

The problem can be solved immediately

````matlab
optimize([F, W],objective);
````

Alternatively, we can first derive a robust version.

````matlab
[Frobust,robustobjective] = robustmodel(F + W,objective);
````

This model does not involve the uncertain variable anymore, and corresponds to the worst-case scenario model. We can now solve the model.

````matlab
optimize(Frobust,robustobjective)
````
