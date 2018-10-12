---
layout: single
category: faq
author_profile: false
excerpt: 
title: Modelling on/off behaviour leads to poor performance
tags:
comments: true
date: '2016-09-17'
sidebar:
  nav:
---

An extremely common mistake beginners make in the development of models is that they have binary variables representing some type of on/off behaviour, and then multiply other variables with this binary to represent things being turned off.

Typical model could for instance be something along the lines of

````matlab
x = sdpvar(N,1);
u = sdpvar(N,1);
d = binvar(N,1);
...
Model = [Model, x(k) == x(k-1) + u(k)*d(k)]
...
````

The product here completely kills this model, as it introduces a nonconvex bilinear equality constraint, thus landing us in a very hard nonconvex nonlinear integer program. A significantly better model is to introduce a new variable to represent the product, and then model the product using simple linear logic. This will keep us in the comfortable world on mixed-integer linear programming (considering these constraints only of course)

````matlab
x = sdpvar(N,1);
u = sdpvar(N,1);
d = binvar(N,1);
w = sdpvar(N,1);
...
Model = [Model, x(k) == x(k-1) + w(k), implies(d(k)==1, w(k)==u(k), implies(d(k)==0, w(k)==0]
...
````

The implication will be converted to linear equalities using the [big-M modelling framework](/tutorials/bigmandconvexhulls), but you can of course do it manually

````matlab
Model = [Model, x(k) == x(k-1) + w(k), -M*d(k) <= w(k) <= M*d(k), -M*(1-d(k)) <= w(k)-u(k) <= M*(1-d(k))]
````

Alternatively, there is a command [binmodel](/commands/binmodel) to use in more complex scenarios if you are too lazy to detect and untangle all the nonlinear products.







