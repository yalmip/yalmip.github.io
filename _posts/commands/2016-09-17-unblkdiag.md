---
layout: single
category: command
author_profile: false
excerpt: ""
title: unblkdiag
tags: [Semidefinite programming]
comments: true
date: '2016-09-17'
sidebar:
  nav: "commands"
---

[unblkdiag] is used to detect and extract block-diagonal terms (typically used to reduce size of a semidefinite program)

### Syntax

````matlab
Y = unblkdiag(X)
````

### Examples

Create a block-diagonal matrix

````matlab
A = sdpvar(2,2);
B = sdpvar(3,3);
C = diag(sdpvar(3,1));
X = blkdiag(A,B,C);
````

Now destroy the block-diagonal structure

````matlab
p = randperm(8);
X = X(p,p);
spy(X)
````

The blocks are easily recovered (note that scalar terms are returned one by one)

````matlab
blocks = unblkdiag(X)
  [3x3 sdpvar]  [1x1 sdpvar]  [2x2 sdpvar]  [1x1 sdpvar]  [1x1 sdpvar]
````

The command is most conveniently used on constraint lists (the function will go through all constraints and try to detect blocked SDP terms)

````matlab
F = X>=0;
F = unblkdiag(F)
+++++++++++++++++++++++++++++++++++++++++++++++++
|   ID|      Constraint|                    Type|
+++++++++++++++++++++++++++++++++++++++++++++++++
|   #1|   Numeric value|   Matrix inequality 3x3|
|   #2|   Numeric value|   Matrix inequality 2x2|
|   #3|   Numeric value|        Element-wise 3x1|
+++++++++++++++++++++++++++++++++++++++++++++++++
````
