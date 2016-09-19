---
title: "Envelope approximations for global optimization"
category: tutorial
author_profile: false
level: 4
tags: [Convex hull, Envelope, Global optimization, BMIBNB]
excerpt: "Outer approximations of function envelopes are the core of the global solver BMIBNB"
layout: single
sidebar:
  nav: "tutorials"
---

The global solver [BMIBNB] is a YALMIP-based implementation of a standard spatial branch-and-bound strategy. If you are unfamiliar with the basic ideas in a branch-and-bound solver, you should try to find an introduction, such as [Lawler1966], before reading this text.

A spatial branch-and-bound algorithm for nonconvex programming typically relies on a few standard steps. 

1. To begin with, in each node, a standard nonlinear solver is applied and a feasible, and hopefully locally optimal solution is computed. This gives an upper bound on the achievable objective (possibly infinite if the solver fails).  The local solver is specified with the option '''bmibnb.uppersolver'''.
2.  As a second step, in each node, a convex relaxation of the model is derived (using the methods described below), and the resulting convex optimization problem is solved (typically a linear program, or if the original problem is a nonconvex semidefinite program, a semidefinite program). This gives a lower bound on the achievable objective. The lower bound solver is specified using the options '''bmibnb.lowersolver'''.
3. Given these lower and upper bounds, a standard branch-and-bound procedure is used to select a branch variable, create two new nodes, branch, prune and navigate the global search.

In addition to these standard steps, a large amount of preprocessing and bound-propagation is performed, both in the root-node and along the branching. This is important in order to obtain stronger linear relaxations. The options controlling this can be found in the introduction to [BMIBNB].

### Linear relaxation for bilinear and quadratic problems

The solver implements a standard relaxation strategy where all bilinear and quadratic terms are replaced with new linear variables. Based on bounds on the variables in the term, constraints are added to relate the new linear variable with the original nonlinear term, in order to make the relaxation as tight as possible. 

Given two variables \\(x\\) and \\(y\\), and lower and upper bounds \\(x_L\\), \\(x_U\\), \\(y_L\\), \\(y_U\\), the following inequalities trivially hold [McCormick1976]


![Bilinear hull]({{ site.url }}/images/xyhull.png){: .center-image }

A linear relaxation of the term \\(xy\\) is obtained by introducing three new variables \\(w_x\\), \\(w_{xy}\\) and \\(w_y\\), and replacing \\(x^2\\), \\(xy\\) and \\(y^2\\) in the constraints above and the original model with the new \\(w\\) variables. A set of linear equalities is then obtained. This procedure is applied to all nonlinear terms.

We can use YALMIP to illustrate this for a simple scalar nonlinearity \\(x^2\\). We generate the nonlinear representation of the generating equations

````matlab
sdpvar x;
xL = -1;
xU = 2;
constraints =[(xU-x)*(x-xL)>=0,(xU-x)*(xU-x)>=0,(x-xL)*(x-xL)>=0]
````

A linear relaxation is obtained by introducing a new variable, and using this variable instead of '''x''''^2^'. This is essentially what happens when you use the option relax when you solve optimization problems. Any monomial term is treated as an independent variable. We use this trick to easily plot the linear relaxation.

````matlab
plot(constraints,[x;x^2],[],[],sdpsettings('relax',1))
t = (-1:0.01:2);
hold on;plot(t,t.^2);
````

![Quadratic hull]({{ site.url }}/images/hullsqr.png){: .center-image }

The horizontal axis is the original variable '''x''', and the polytope shows the feasible region for the relaxed variable. Clearly, the approximation is perfect at the bounds, but fairly poor in the middle. It should be stated that positivity bounds on quadratic terms automatically are appended to relaxation.

By tightening the bounds during the branching process, the linear relaxation will become increasingly better. As an example, if the solver branches on the '''x''' variable at 0.5, two new nodes would be created with stronger relaxations.

````matlab
sdpvar x;
xL1 = -1;
xU1 = 0.5;
xL2 = 0.5;
xU2 = 2;

constraints1 =[(xU1-x)*(x-xL1)>=0,(xU1-x)*(xU1-x)>=0,(x-xL1)*(x-xL1)>=0]
constraints2 =[(xU2-x)*(x-xL2)>=0,(xU2-x)*(xU2-x)>=0,(x-xL2)*(x-xL2)>=0]

clf
plot(constraints1,[x;x^2],[],[],sdpsettings('relax',1))
hold on
plot(constraints2,[x;x^2],[],[],sdpsettings('relax',1))
t = (-1:0.01:2);
hold on;plot(t,t.^2);
````

![Quadratic hull]({{ site.url }}/images/xyhull2.png){: .center-image }

Notice that the algorithm requires explicit bounds on the variables. If YALMIP fails to extract any explicit bounds from the model in the root-node, the code will terminate and an error message will be displayed.

### Linear relaxation for polynomial problems

Polynomials problems are treated by simply converting them to bilinear problems, by introducing additional variables and constraints. As an example, the term \\(x^2y^2\\) can be written as \\(uv\\) with the new bilinear constraints \\(u=x^2\\) and \\(v=y^2\\). Once a bilinear model has been obtained, standard bilinear relaxations can be used.

### Linear relaxation for general problems

The global solver is mainly intended for polynomial problems, but does actually support also general nonlinear functions. Similar to the polynomial case, every nonlinear scalar term \\(f(x)\\) is replaced with a new variable \\(w\\), and linear inequalities on \\(x\\) and \\(w\\) are introduced to ensure that \\(w\\) approximates \\(f(x)\\) well. 

As an example, for the **sqrtm** function, the following cut is used internally

````matlab
xL = 0.1;
xU = 3;
Aw = [1; 1; -1];
xM = (xL + xU)/2;
b = [sqrt(xL) - xL/(2*sqrt(xL));
     sqrt(xU) - xU/(2*sqrt(xU));
     sqrt(xU)*(xL)/(xU-xL) -  sqrt(xL)*(xU)/(xU-xL)];

Ax = [-1/sqrt(4*xL);
      -1/sqrt(4*xU);
       sqrt(xU)/(xU-xL) - sqrt(xL)/(xU-xL)];
sdpvar x w
figure
plot([Ax*x + Aw*w <= b],[x;w]);
t = (xL:0.01:xU);
hold on;plot(t,sqrt(t));
````

![Quadratic hull]({{ site.url }}/images/hullsqrtm.png){: .center-image }

Explicit representations of the linear relaxation are implemented for most nonlinear operators, such as **exp**, **log** and **sqrtm**. For some other [evaluation based] and [sdpfun] based nonlinear operators, a sampling strategy is used to derive the linear relaxation (of course, exact convex envelopes can be developed if someone really needs it). As an example, the following code computes (an approximation of) a convex envelope relaxation of **sin**.

````matlab
xL = 0;
xU = 3*pi/2;
z = linspace(xL,xU,100);
fz = sin(z);
k1 = max((fz(2:end)-fz(1))./(z(2:end)-xL))+1e-3;
k2 = min((fz(2:end)-fz(1))./(z(2:end)-xL))-1e-3;
k3 = min((fz(1:end-1)-fz(end))./(z(1:end-1)-xU))+1e-3;
k4 = max((fz(1:end-1)-fz(end))./(z(1:end-1)-xU))-1e-3;
Ax = [-k1;k2;-k3;k4];
Aw = [1;-1;1;-1];
b =  [k1*(-z(1)) + fz(1);-(k2*(-z(1)) + fz(1));k3*(-z(end)) + fz(end);-(k4*(-z(end)) + fz(end))];
sdpvar x w
figure
plot([Ax*x + Aw*w <= b],[x;w]);
t = (xL:0.01:xU);
hold on;plot(t,sin(t));
````

![Quadratic hull]({{ site.url }}/images/hullsin.png){: .center-image }
