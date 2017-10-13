---
layout: single
permalink: /modellingif
excerpt: "Untangle that messy expression"
title: "Modelling if-else-end statements"
tags: [Mixed-integer linear programming, Logic programming]
comments: true
date: '2017-10-13'
---

YALMIP supports complex models by overloading standard operators in MATLAB. One common issue though that many users struggle with is modelling of **if** statements. The object-oriented overloading of operators in MATLAB does not support overloading of programming constructs, i.e., **if x**, **elseif x**, **switch x**, **for x=** where **x** is an [sdpvar](/command/sdpvar) will not work.

Hence, this logic has to be modelled manually, and the trick to do this is to apply the [implies operator](/command/logic) in a structured fashion.

## A worked example

Consider the problem of solving a form of linear regression where we want to minimize a \\( f(Ax-b)\\) where we want to use the penalty 

THe naive way to write the model would be

if e <= verylow
 f = 1;
elseif e >= verylow && e <= low
 f = 1-x;
elseif e>=low && e <= high
 f = e^2;
elseif e>=high && e <= veryhigh
 f = 1+x;
else 
 f = 2;
end 
 
This code will not run, and MATLAB will raise an error, as you are you are using  an sdpvar in the if construct. What we have to do is to translate this to simple combinatorial cases, and implement that using binary variables

### Enumerate the possible cases

For the cleanest possible code, you should think through what are the possible cases in the most simplified model possible, count these, and introduce a binary variable for every case. One should also strive for a descption where the possible sets are disjunctive in the sense that exactly one case can and should occur.

In this case, there are obviously 5 possible cases clearly dividing the feasible space into 5 regions. Hence, we introduce 5 binary variables, and create a model where each of these binary variables forces the decision variable to be in the associated region, and the corresponding expression on the cost function to hold

cases = binvar(5,1);
Model = [sum(cases) == 1,
implies(cases(1), [            e <= verylow,   f == 2]);
implies(cases(2), [ verylow <= e <= low,f == 1-x]);
implies(cases(3), [     low <= e <= high, f == e^2]);
implies(cases(4), [    high <= e <= veryhigh, f == 1+x]);
implies(cases(5), [veryhigh <= e, f == 2])];










 
