---
category: command
excerpt: ""
title: savecplexlp
tags: [Export and import]
date: '2016-09-17'
sidebar:
  nav: "commands"
---

[savecplexlp](/command/savecplexlp) exports a YALMIP model to a cplex lp format file

### Syntax

````matlab
saveampl(F,h,filename)
````

### Examples

The command enables export of simple YALMIP models to CPLEX LP data files

````matlab
x = sdpvar(3,1);
A = randn(5,3);
b = randn(5,1);
c = randn(3,1);
F = [A*x <= b, integer(x(2:3))];
saveampl(F,c'*x,'mymodel');
````

### Comments

This function has not been tested to any extent and is probably severely limited.
