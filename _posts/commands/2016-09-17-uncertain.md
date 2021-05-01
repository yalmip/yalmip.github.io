---
category: command
excerpt: ""
title: uncertain
tags: [Robust optimization]
date: '2016-09-17'
sidebar:
  nav: "commands"
---

[uncertain](/command/uncertain) is used to declare a set of variables as uncertain, or to simultaneously add a set of constraints to the uncertainty set, and declare all involved variables as uncertain. It can also be used for attaching random distributions and samplers to a variable

### Syntax

````matlab
F = uncertain(w) % w variable
F = uncertain(W) % w constraint
F = uncertain(w,distribution)
````

### Examples

A simple [robust optimization](/tutorials/robustoptimization) problem can be implemented as

````matlab
sdpvar x w
F = [x+w <= 1];
W = [-0.5 <= w <= 0.5, uncertain(w)];
objective = -x;
optimize([F, W],objective)
````
or

````matlab
F = [x+w <= 1];
W = [uncertain(-0.5 <= w <= 0.5)];
objective = -x;
optimize([F, W],objective)
````

To specify random uncertainties for use in [optimizers](comand/optimie) and [sample](/command/sample), you specify the distribution, and all distribution parameters following the syntax in the RANDOM command in the Statistics Toolbox
 
 ````matlab
sdpvar x w
F = [x + w <= 1, uncertain(w, 'uniform',0,1)];
P = optimizer([F,W,uncertain(w)],-x,[],w,x)
S = sample(P,10); % Sample ten instances and concatenate models
S([])             % Solve and return optimal x
````
  
Alternatively, you can specify a function handle which generates samples. YALMIP will always send a trailing argument with dimensions

````matlab 
F = [x + w <= 1, uncertain(w,@mysampler,myarguments1,...)];
````

The standard random case above would thus be recovered with

````matlab
F = [x + w <= 1, uncertain(w,@random,0,1)];
```` 

