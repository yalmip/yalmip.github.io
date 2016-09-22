---
layout: single
category: command
author_profile: false
excerpt: ""
title: uncertain
tags: [Robust optimization]
comments: true
date: '2016-09-17'
sidebar:
  nav: "commands"
---

[uncertain](/command/uncertain) is used to declare a set of variables as uncertain, or to simultaneously add a set of constraints to the uncertainty set, and the involved variables as uncertain.

### Syntax

````matlab
sdpvar x w
F = [x+w <= 1];
W = [-0.5 <= w <= 0.5, uncertain(w)];
objective = -x;
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
