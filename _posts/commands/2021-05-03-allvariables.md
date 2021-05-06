---
category: command
excerpt: ""
title: allvariables
tags: []
date: '2021-05-03'
sidebar:
  nav: "commands"
---

[allvariables](/command/allvariables) creates a vector with variables used in a set of objects

### Syntax  

````matlab
y = allvariables(A,B,...)
````

### Examples

Create a set of constraints and an objective, and then create an new objective function which adds a penalty on all involved variables

````matlab
% My original model
x = sdpvar(3,1);y = sdpvar(1);
Model = [-1 <= x <=1, y == sum(x)];
Obj = sum(x) + y^2;

% No, I want to solve with a regularization on all variables
Obj = Obj + 0.01*norm(allvariables(Model,Obj))
````

### Comments

The command is a utility replacing repated application of [depends](/command/depends) followed by [recover](/command/recover), i.e. it simplifies patterns of the type

````matlab
x1 = depends(A);x2 = depends(B);
y = recover(unique([x1(:);x2(:)]));
````


