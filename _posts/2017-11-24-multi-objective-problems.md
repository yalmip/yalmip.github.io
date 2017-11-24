---
layout: single
permalink: /multiobjective
excerpt: "How do I create a cheap Ferrari?"
title: "Multi-objective problems in YALMIP"
tags: [Multiobjective programming, Portfolio optimization]
comments: true
date: '2017-11-24'
---

A common question is how one can solve [multi-objective problems](https://en.wikipedia.org/wiki/Multi-objective_optimization) using YALMIP. The standard answer is that you cannot solve these using YALMIP. A more detailed answer is that you cannot solve these problem until you have sorted out what you actually mean when you talk about a solution to a multi-objective problem, as there is no single solution but a set of solutions, and finding this set can be done using various strategies.

At the core, there is typically a fundamental misconception about multi-objective optimization, in the sense that it would be possible to magically compute **a** solution which optimizes several competing objective. Of course, it is impossible to design a car which is as light as possible, as cheap as possible, as fast as possible, and as durable as possible, all at the same time. In the end, the solution to the obviously multi-objective task of designing a car, will be a compromise. Multi-objective optimization is about finding **the set** of non-bad compromises, which is called the Pareto-optimal solutions.

So, the answer to the question is

* No, you cannot compute **the** solution to the multi-objective problem, as there is no such thing.
* No, YALMIP does not interface any multi-objective solver to compute the pareto-optimal solution set.
* Yes, of course you can compute solutions to the multi-objective problem by simply implementing a multi-objective algorithm using any standard solver to compute solutions to the optimization problems that arise.

## Scalarizing a the multi-objective

A simple way to attack a multi-objective problem is to scalarize the objective functions, i.e. introduce a single objective which represents a compromise of the competing objectives. 

Consider the [portfolio example](/example/portfolio)

````matlab
n = 10;
S = randn(n);S = S*S'/1000; % Covariance
mu  = rand(1,n)/100;        % Expected return       
w = sdpvar(n,1);
mutarget = mean(mu);
Constraints = [sum(w) == 1, w>=0];

Variance = w'*S*w;
Return = -mu*w;
````

In this problem, we have two competing objectives, minimizing the variance of the portfolio return, and maximizing the expected return. A simple scalarization of this problem could simply be a linear combination of them (note that we have negated the return above as we want to maximize it).

````matlab
optimize(Constraints, 0.5*Variance + 0.5*Return);
````

Why 0.5? Well, that is precisely the problem. There is no clear-cut answer on that. It depends on what you want to achieve, i.e., which compromise you are interested in.

Hence, what one typically wants to do then is to solve the problem for a range of scalarization compromises, and try to get a feeling for the character of the solutions, and use some logic to pick the best compromise.

````matlab
clf
hold on
for t = 0:0.05:1
  optimize(Constraints, (1-t)*Variance + t*Return);
  plot(value(Variance),value(-Return),'*')
  drawnow
end
````

We can speed this up by using the possibility of [multiple solutions computed in one shot](/multiplesolutions). This requires linear objectives though so we make a simple epigraph reformulation of the objective. Note that this moves us from a quadratic program to a second-order cone program. 

````matlab
sdpvar v
Constraints = [sum(w) == 1, w>=0,  w'*S*w <= v];
Variance = v;
Return = -mu*w;

clf
hold on
t = 0:0.05:1
Objectives = (1-t)*Variance + t*Return;
optimize(Constraints, Objectives);
for i = 1:length(t)
 selectsolution(i);
 plot(value(w'*S*w),value(-Return),'*')
 drawnow
end
````

An more general approach is to implement the strategy using an [optimizer object](/comands/optimizer). Note that we return the value of the variance computed from the allocation, instead of the simple epigraph variable. The reason is that the epigraph variable is not used when **t = 1** and thus the epigraph variable and the expression it bounds are not the same in that border case. Also note that the solver has to be specified, in this case we pick [MOSEK](/solvers/mosek)

````matlab
sdpvar v
Constraints = [sum(w) == 1, w>=0,  w'*S*w <= v];
Variance = v;
Return = -mu*w;
sdpvar t
Solver = optimizer(Constraints,(1-t)*Variance + t*Return,sdpsettings('solver','mosek'),t,{w'*S*w,Return})

clf
hold on
for t = 0:0.05:1
 sol = Solver(t);
 plot(sol{1},-sol{2},'*')
 drawnow
end
````

Even faster and more compact!

````matlab
t = 0:0.05:1
sol = Solver(t);
plot(sol{1},-sol{2},'*')  
````

So to summarize: yes you can implement any multi-objective solver strategy you want as long as you can model it using YALMIP.
