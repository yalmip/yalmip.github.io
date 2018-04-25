---
layout: single
permalink: /parameterizeduncertainty
excerpt: "Uncertainty descriptions can only involve uncertain variables, so how can they be parameterized?"
title: "Parameterizing the uncertainty set in robust optimization"
tags: [Robust optimization, Uncertainty]
comments: true
published: true
date: 2018-04-25
---

In this quick post, we look at the problem of having parameterized uncertainty sets, and how these can be dealt with by reparameterizing the uncertainty.

The type of models YALMIP works with in [robust optimization](/tutorial/robustoptimization) is assumed to of the form 

$$
\begin{aligned}
\text{min}_x \text{max}_w f(x,w) & \\
\text{subject to}~ & g(x,w) \leq 0~ \forall~ \{w ~| ~h(w) \leq 0 \}
\end{aligned}
$$

However, in some cases it might be that the uncertainty set is parameterized, and the parameters defining the geometry are seen as decision variables. A typical case would be that the size of the set is parameter, and we wish to compute a robust solution which is some compromise between the level of robustness and conservativity. YALMIP does not support this, as YALMIP cannot distinguish between normal decision variables, and parameters in uncertainty sets. If a constraint is defined using both uncertain variables and variables which are not declared as uncertain, it will be considered to be an uncertain constraint, and not a definition of an uncertainty set.

## Reparameterizing the uncertainty

As an example, let us consider the problem of computing the Chebychev L1-ball of a polytope, i.e., the largest possible romb that can be placed inside a given polytope. This can be formulated as the robust optimization problem

$$
\begin{aligned}
\text{max}_{z_c,r}~r & \\
\text{subject to}~ & Az \leq b ~\forall~ \{|z-z_c|_1 \leq r\}
\end{aligned}
$$

Here, \\(z\\) is the uncertain variable, and the center of the ball \\(z_c\\) and the size of the ball \\(r\\) are parameters in the uncertainty description.

The incorrect YALMIP model would be

````matlab
A = randn(10,2);b = rand(10,1)*10;

z = sdpvar(2,1);
zc = sdpvar(2,1);
r = sdpvar(1);

UncertainConstraint = [A*z <= b];
UncertaintyModel = [norm(z-zc,1) <= r];
optimize([UncertainConstraint,UncertaintyModel,uncertain(z)],-r)
````

This model will fail, since YALMIP cannot find the intended uncertainty desciption here, as there is no constraint which only involves uncertain variables. Adding \\(r\\) and \\(z_c\\) to the list of uncertain variables does not make sense either, as they are not uncertain, but simply not decided yet.

The way to model this setup is to think of a normalized uncertainty, and then write the uncertainty in terms of this normalized uncertainty. In our case, we can see the uncertainty as a scaled and translated uncertainty.

````matlab
w = sdpvar(2,1);
zc = sdpvar(2,1);
r = sdpvar(1);

z = r*w + zc;
UncertainConstraint = [A*z <= b];
UncertaintyModel = [norm(w,1) <= 1];
optimize([UncertainConstraint,UncertaintyModel,uncertain(z)],-r)
````

This is a very simple model, and YALMIP derives the robust counterpart and computes the largest inscribed L1-ball.

````matlab
plot(A*w <= b,w,'red');hold on;
plot(norm(w - value(zc),1)<= value(r),w,'yellow')
````

![L1 Chebychev ball]({{ site.url }}/images/L1cheby.png){: .center-image }
