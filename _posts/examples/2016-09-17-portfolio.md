---
layout: single
permalink: /examples/portfolio
type: example
author_profile: false
excerpt: "Markowitz classical portfoilos and beyond via integer programming."
title: Portfolio optimization
tags: [Portfolio optimization, Finance, Quadratic programming, Integer programming]
comments: true
date: '2016-09-16'
header:
  teaser: "portfolio.png"
sidebar:
  nav: "examples"
---

### Standard Markowitz portfolio

The standard [Markowitz mean-variance portfolio problem](http://en.wikipedia.org/wiki/Modern_portfolio_theory) is to select assets relative investements \\(w\\)) to minimize the variance \\(w^TSw\\)) of the portfolio profit while giving a specified expected return \\(\mu^{\star}\\), given historical data of mean returns \\(\mu\\) and covariance \\S\\) of stock returns.

Let us begin by defining some random data with 10 candidate assets.

````matlab
n = 10;
S = randn(n);S = S*S'/1000; % Covariance
mu  = rand(1,n)/100;        % Expected return       
````

We introduce a vector **w** to model the position in each asset, and solve the standard Markowitz portfolio problem (no short positions) and aim for an expected return equal to the average historical return of all assets.

````matlab
w = sdpvar(n,1);
mutarget = mean(mu);
F = [sum(w) == 1, w>=0, mu*w == mutarget];
optimize(F,w'*S*w)
value(w)
````

An alternative approach is to limit the variance, and maximize the expected return. Let us maximize the return while constraining the variance to be less than the variance for a  portfolio with equal positions in all assets (this model leads to a quadratically constrained problem, hence you need a QCQP or SOCP capable solver such as [SEDUMI], [SDPT3], [GUROBI], [MOSEK], or [CPLEX])

````matlab
F = [sum(w) == 1, w>=0];
F = F + [w'*S*w < sum(sum(S))/n^2];
optimize(F,-mu*w)
value(w)
````

Yet another proposal is to maximize the Sharpe ratio \\(\frac{w^T(\mu-\mu_0)}{\sqrt{w^TSw}}\\) where \\(\mu_0\\) is the return of a risk-free asset, i.e., the excess return per unit standard deviation. A first sight, this is terribly nonlinear and can indeed also be non-convex. However, it can easily be converted to a quadratic program. In principle, we note that the normalization that the weights should sum to 1 is rather arbitrary, we just want it to be non-zero and the important result is the relative sizes. We can just as well use a new set of weights \\(z\\) with the constraint that \\(z^T(\mu-\mu_0)\\). With this, our objective is maximization of \\(\frac{1}{\sqrt{z^TSz}}\\), i.e., minimization of \\(z^TSz\\)

````matlab
mu0 = 0.001;
z = sdpvar(n,1);
optimize([z'*(mu(:)-mu0)==1, z>=0],z'*S*z)
w = value(z)/sum(value(z))
````

The solutions to the problems above typically leads to a portfolio with shares in most assets. This is typically not wanted, since it leads to increased transaction and administration costs.

### Cardinality & structurally constrained portfolios

An alternative is to only allow a limited number of non-zero positions. This is easily modeled using the [nnz] operator, and leads to a mixed integer quadratic programming problem. Let us limit the number of positions to 4 (since [nnz] requires a MILP model, we explicitly upper bound the variable **w** to help the [Tutorials.Big-M  big-M] reformulation). Please note that cardinality constrained portfolio selection yields extremely hard problems.

````matlab
mutarget = mean(mu);
F = [sum(w) == 1, 1>=w>=0, mu*w == mutarget];
F = F + [nnz(w) <= 4];
optimize(F,w'*S*w)
value(w)
````

To ensure a balanced portfolio, constraints on smallest and largest fraction could be useful. We can do this by hand using the following model, where we aim for a portfolio where no asset accounts for more than 80%, but at the same time no non-zero positions are smaller than 10%.

````matlab
d = binvar(n,1);           % models if variable is nonzero
F = [sum(w) == 1, 1>=w>=0, mu*w == mutarget];
F = F + [sum(d) <= 4]   % At most 4 positions
F = F + [w <= d];       % If d==0 then w = 0
F = F + [w >= 0.1*d];   % If d==1 then w >= 0.1
F = F + [w <= 0.8];     % No position >= 0.8
optimize(F,w'*S*w)
value(w)
````

An equivalent model can be obtained by using the command [semivar]. The drawback with the following code is that it can be slightly less efficient since it introduce separate binary variables for the cardinality constraint and the semi-continuous position vector. An efficient pre-solve in the integer solver should be able to detect these redundant variables though.

````matlab
w = semivar(n,1);      % Either 0...
F = [0.1 <= w <= 0.8]; % ...or between bounds
F = [F, sum(w) == 1, mu*w == mutarget];
F = [F, nnz(w) <= 4];      % At most 4 positions
optimize(F,w'*S*w)
value(w)
````

The following portfolio only allows fixed sizes of the positions, while aiming for the target return

````matlab
w = sdpvar(n,1);
F = [sum(w) == 1, 1>=w>=0];
F = [F, ismember(w,0:0.1:1)];  % Sizes in steps of 10%
optimize(F,w'*S*w + (mu*w - mutarget)^2)
value(w)
````

### The efficient frontier

A school-book example of parametric optimization is the efficient frontier in the Markowitz portfolio. This is the lowest possible variance \\(w'^TSw\\) achievable, when striving for a particular profit.

!### Repeated solutions using the optimizer command

Solving many problems with only a small change in the setup can in some cases be done efficiently using the [optimizer] command. This command allows us to create an object which takes the target return as input, and returns the variance and portfolio assignments as outputs. By using the [optimizer] command instead of explicitly setting up a new problem and calling [optimize], the overhead can be reduced drastically.

````matlab
w = sdpvar(n,1);
sdpvar mutarget
F = [sum(w) == 1, w>=0, mu*w == mutarget];
Portfolio = optimizer(F,w'*S*w,sdpsettings('verbose',0),mutarget,[w'*S*w;w]);
````

Optimal portfolios can now easily be computed in one shot.

````matlab
targets = linspace(min(mu),max(mu),100);
solutions = Portfolio{targets};
plot(targets,solutions(1,:))
````

### Multiparametric approach

We can alternatively compute and plot this curve, if [MPT] is installed.

````matlab
w = sdpvar(n,1);
sdpvar mutarget
F = [sum(w) == 1, w>=0, mu*w == mutarget];
[sol,dgn,aux,Valuefcn,woptimal] = solvemp(F,w'*S*w,[],mutarget);
plot(Valuefcn);
````

![polytopicsystem]({{ site.url }}/images/portfolio.png){: .center-image }

The minimum risk portfolio and the associated variance at the target profit that we computed in the beginning of this example can be extracted from the parametric solution

````matlab
assign(mutarget,mean(mu))
value(woptimal)
value(Valuefcn)
````
