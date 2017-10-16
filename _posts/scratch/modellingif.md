---
layout: single
permalink: /modellingif
excerpt: "Untangle that messy expression"
title: "Modelling if-else-end statements"
tags: [Mixed-integer linear programming, Logic programming]
comments: true
date: '2017-10-13'
---

YALMIP supports complex models by overloading standard operators in MATLAB. One common issue though that many users struggle with is modelling of **if** statements. The object-oriented overloading of operators in MATLAB does not support overloading of programming constructs, i.e., **if x**, **elseif x**, **switch x**, **for x=** where **x** involves [sdpvar](/command/sdpvar) variables will not work.

Hence, this logic has to be modelled manually, and the recommended approach to do this is to apply the [implies operator](/command/logic) in a structured fashion.

## An artificial regression problem

Consider a problem where we wish to do regression with less sensitivity to outliers by using a non-convex penalty. For small residuals, we have a continuous penalty, medium-sized residuals are penalized with a linear cost, and really large residuals have constant penalty.

For a scalar argument, the function can conceptually be written using the following evaluation in MATLAB

````matlab
if e <= -5
 f = 1;
elseif e >= -5 && e <= -1
 f = 1-x;
elseif e>=-1 && e <= 1
 f = e^2;
elseif e>=1 && e <= 5
 f = 1+x;
elseif e >= 5 
 f = 2;
end 
````

This code will not run, and MATLAB will raise an error, as you are you are using an [sdpvar](/command/sdpvar) in the **if** construct. What we have to do is to translate this to simple combinatorial cases, and implement that using binary variables.

### Enumerate the possible cases

For the cleanest possible code, you should think through what are the possible cases in the most simplified model possible. With simple, we mean no **else** statements, as few logical combinations as possible, and a set of conditions where only one thing can or should happen. In our case, a clean version would be

````matlab
if e <= verylow
 f = 1;
end;
 
if e >= verylow && e <= low
 f = 1-x;
end

if e>=low && e <= high
 f = e^2;
end
 
if e>=high && e <= veryhigh
 f = 1+x;
end

if e >= veryhigh
 f = 2;
end 
````

In this case, there are obviously 5 possible cases dividing the feasible space into 5 regions. Hence, we introduce 5 binary variables, and create a model where each of these binary variables forces the decision variable to be in the associated region, and the corresponding expression on the cost function to hold. Note that the expression used in the **if**-statement, and the resulting action, both are moved to a list of constraint.

````matlab
cases = binvar(5,1);
Model = [sum(cases) == 1,
implies(cases(1), [            e <= verylow,   f == 2]);
implies(cases(2), [ verylow <= e <= low,f == 1-x]);
implies(cases(3), [     low <= e <= high, f == e^2]);
implies(cases(4), [    high <= e <= veryhigh, f == 1+x]);
implies(cases(5), [veryhigh <= e, f == 2])];
````

### A more natural model which might lead to problems

It should be said, that a lazy approach to create a thoretically equivalent model is

````matlab
Model = [implies(e <= verylow,   f == 2);
         implies(verylow <= e <= low,f == 1-x);
         implies(low <= e <= high, f == e^2);
         implies(high <= e <= veryhigh, f == 1+x);
         implies(veryhigh <= e, f == 2)];
````

Although this looks much easier, it hides important structure that can cause problems. As all those constraints are added independently, there is nothing in the model which explicitly forces one of the cases to hold. Due to numerical tolerances in solvers, it might be the case that the optimal **e** ends up at a point on the border between to regions where the solver manages to violate a condition every so slightly and renders all conditions inactive thus making **f** undefined. Hence, it is always recommended to explicitly define the disjunctive nature of the model by introducing a binary variables which must activate one condtion.
