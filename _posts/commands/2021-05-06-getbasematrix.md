---
category: command
excerpt: "Etract the basis of an expression for variable index"
title: getbasematrix
tags: [Low-level]
date: '2021-05-06'
sidebar:
  nav: "commands"
---

[getbase](/command/getbasematrix) returns the full basis for a particular variable index in an [sdpvar](/command/sdpvar) expression.

## Syntax

````matlab
B = getbasematrix(x,index)
````

## Examples

Expressions are are built-up from internal variable indicies ([getvariables](/command/getvariables)) and a basis. We can extract the full basis with respect to a particular variable index

````matlab
yalmip('clear')
x = sdpvar(1);
y = sdpvar(1);
z = [1 2*x;3*y 4*x+5*y + 6*x^2];

full(getbasematrix(z,getvariables(y)))
ans =

     0     0
     3     5

full(getbasematrix(z,getvariables(x^2)))

ans =

     0     0
     0     6
     
full(getbasematrix(z,0))     

ans =

     1     0
     0     0
````

To extract the whole basis use [getbase](/command/getbase).
