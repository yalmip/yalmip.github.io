---
layout: single
permalink: /equalityinuncertainty
excerpt: "= ≠ =="
title: "Equalities with uncertainty"
tags: [Robust optimization, Uncertainty]
comments: true
published: true
date: 2018-09-11
---

If I had a million dollars for every time I've had to explain this, I would definitely not write this post. 

Trying to develop [robust optimization](/tutorial/robustoptimization) models with [uncertainty](/command/uncertain/) appearing in equality constraints is a common mistake, and the reason people do this mistake is almost always due to failure to

 - understand the difference between a **decision variable** and an **expression involving decision variables**
 
 - understand the difference between **an equality constraint ==** and an **assignment =**
  
 - underastand the difference between a **decison variable** and a **policy or decision rule**.

### Buying a horse the old way

To understand the problem, we work with the following problem.

Alice and Bob are going to the market to buy a horse. They will pay the horse by trading with sheep. They do not know the current market price **p** of a horse, all they know is that it cost somewhere between 100 and 200 sheep. Alice and Bob will bring the sheep in their trucks. Alice will bring **a** sheep in her truck, while  Bob will bring **b** sheep. They have been told by their manager that they can under no circumstances bring any sheep back to the farm, and they can leave no sheep at the market but all sheep must be used to pay for the horse, and they must pay the market price and nothing else. Alice truck can pack 200 sheep while Bob only can carry 120 sheep.
{: .notice--success}

To figure out how many sheep they should bring in their trucks when going to the market, they must thus solve the uncertain feasibility problem

$$
0 \leq a \leq  200, 0 \leq b \leq 120,  a + b = p~\forall~100 \leq p \leq 200
$$

The corresponding YALMIP model would be

````matlab
sdpvar a b p
Model = [0 <= a <= 200, 0 <= b <= 120, a + b == p, 100 <= p <= 200, uncertain(p)]
````

It should be pretty obvious that this is a nonsense problem. It is completely impossible to select the fixed decisions **a** and **b** at the farm, go the market and figure out the price, and magically have the correct number. Indeed, if you try to solve the optimization problem, it will be infeasible.

Let us untangle what has gone wrong. The first issue is that the setup simply doesn't make sense in real-life, which translates to the incorrect use of an equality ==. So what could the manager have meant that they should do, and how does that translate to the model.

It could be that the manager meant that Alice first should load her truck with **a** sheep (make a fixed decision), go to the market and check the price, and then call Bob and have him bring the remaining sheep. This is a causal structure where Bobs decision **b** really isn't a decision, but a simple function of Alice decision and the uncertainty, **b = p-d**. An **incorrect** model of this would be

````matlab
sdpvar a b p
Model = [0 <= a <= 200, 0 <= b <= 120, b == p -a, 100 <= p <= 200, uncertain(p)]
````

The problem with this model is that we are still saying that Bobs load is a fixed decision (since it is its own decision variable), while we really only want it to be a function of Alice load and the market price. We thus do not want to model an equality between two independent decision variables and an uncertain price, but simply define the linear map that arise due to the causal (time-dependent order) structure in the problem. The correct model would thus be

````matlab
sdpvar a p
b = p-a
Model = [0 <= a <= 200, 0 <= b <= 120, 100 <= p <= 200, uncertain(p)]
````
In this model there is only 1 decision variable (**a**). Bobs load is simply an assingment from fixed decisions and uncertainties.

Another version could be that the manager meant that Alice and Bob actually should pick up the phone and ask what the price is, and then act accordingly. Once again, that simply means we want to define a fixed map from the price to the loads, and we no longer have any decision variables at all. One such decision rule, or policy, is that they always use \(a = \frac{p}{2}\) and \(b = \frac{p}{2}\). In other words, our model would be

````matlab
sdpvar p
a = p/2
b = p/2
Model = [0 <= a <= 200, 0 <= b <= 120, 100 <= p <= 200, uncertain(p)]
````

This model is a bit too simple (we have no decision variables!) but it illustrates the idea of a policy compared to a decision. An improvement which could be useful in a more complex scenario is to parameterize the policy. The manager thinks Alice and Bob are too dumb to load cleverly once they know the price (Alice truck is bigger so perhaps they should load more Sheep in her trcuk). Hence, he wants to create a function that they can use to select the number of cheap to load. One such policy is a linear decision rule \(a = t_0 + t_1p, b = s_0 + s_1p\). As long as \(t_0+s_0 = 0\) and \(t_1+s_1 = 1\), they will bring the correct amount. Here, we thus decide upon the decision rule parameters before knowing the price of the horse, but once the price is known, we have a method to distribute the load.

````matlab
sdpvar p t0 t1 s0 s1
a = t0 + t1*p;
b = s0 + s1*p;
Model = [0 <= a <= 200, 0 <= b <= 120, t0+s0 == 0, t1+s1 == 1, 100 <= p <= 200, uncertain(p)]
````

We explictly derived the condition necessary on the policy parameters for it to be correct. This can be done automatically by exploiting the fact that an equality involving uncertainties make no sense. The following model will work equivalently

````matlab
sdpvar p t0 t1 s0 s1
a = t0 + t1*p;
b = s0 + s1*p;
Model = [0 <= a <= 200, 0 <= b <= 120, a + b == p, 100 <= p <= 200, uncertain(p)]
````

So how can this work, since we've just learned that you cannot have uncertainties in equalities? When YALMIP finds the equality involving the uncertainty, it will derive the conditions required for the equality to be reasonable, i.e., it will derive the conditions necessary for the decision variables in order to completely eliminate any uncertainty in the equality. In this case, it will see the equality \(t_0 + s_0 + (t_1 + s_1)p = p\), and thus the only was this can be feasible for uncertain \(p\) is that \(t_1+s_1=0\), and the remaining term in the equality then says \(t_0 + s_0 = 0\).





