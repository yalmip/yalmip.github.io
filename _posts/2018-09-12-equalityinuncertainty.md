---
layout: single
permalink: /equalityinuncertainty
excerpt: "= ≠ ==. Horse trading and warehouse logistics"
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
In this model there is only 1 decision variable (**a**). Bobs load is simply an assignment from fixed decisions and uncertainties. We've introduced an intermediate placeholder for our convenience, and what YALMIP sees here is really

````matlab
sdpvar a p
Model = [0 <= a <= 200, 0 <=  p-a <= 120, 100 <= p <= 200, uncertain(p)]
````

Another version could be that the manager meant that Alice and Bob actually should pick up the phone and ask what the price is, and then act accordingly. Once again, that simply means we want to define a fixed map from the price to the loads, and we no longer have any decision variables at all. One such decision rule, or policy, is that they always use \\(a = \frac{p}{2}\\) and \\(b = \frac{p}{2}\\). In other words, our model would be

````matlab
sdpvar p
a = p/2
b = p/2
Model = [0 <= a <= 200, 0 <= b <= 120, 100 <= p <= 200, uncertain(p)]
````

This model is a bit too simple (we have no decision variables!) but it illustrates the idea of a policy compared to a decision. An improvement which could be useful in a more complex scenario is to parameterize the policy. The manager thinks Alice and Bob are too dumb to load cleverly once they know the price (Alice truck is bigger so perhaps they should load more Sheep in her trcuk). Hence, he wants to create a function that they can use to select the number of cheap to load. One such policy is a linear decision rule \\(a = t_0 + t_1p, b = s_0 + s_1p\\). As long as \\(t_0+s_0 = 0\\) and \\(t_1+s_1 = 1\\), they will bring the correct amount. Here, we thus decide upon the decision rule parameters before knowing the price of the horse, but once the price is known, we have a method to distribute the load.

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

So how can this work, since we've just learned that you cannot have uncertainties in equalities? When YALMIP finds the equality involving the uncertainty, it will derive the conditions required for the equality to be reasonable, i.e., it will derive the conditions necessary for the decision variables in order to completely eliminate any uncertainty in the equality. In this case, it will see the equality \\(t_0 + s_0 + (t_1 + s_1)p = p\\), and thus the only was this can be feasible for uncertain \\(p\\) is that \\(t_1+s_1=0\\), and the remaining term in the equality then says \\(t_0 + s_0 = 0\\).


### A warehouse logistics problem

In this second example, we address precisely the same mistakes and correct them in the same way. 

Alice manages a warehouse selling goods. The stock of goods available in the warehouse on the morning of day \\(i\\) is \\(w_i\\). Everyday, she sells \\(s_i\\) items but this quantity is fluctuating so all we know a-priori is that \\(200 \leq s_i \leq 1000\\). She also receives delivery of supplies every day. The delivery is not instant but arrives in the morning two days after the order was made (order in the evening of day \\(i\\) arrives in morning of day \\(i+2\\)). We call the order \\(u_i\\). We cannot remove or ship back inventory, i.e. \\(u_i \geq 0\\). Some customers are not happy with the product, so 10% of all sold items are returned two days after it was sold, for simplicity this is assumed to occur in the morning. Collecting all information, we have the dynamical system 
$$
w_i = w_{i-1} - s_{i-1} + u_{i-2} + 0.1s_{i-2}
$$
The warehouse can only stock 8000 items, and we must always have enough in stock to be able to sell to all prospective customers. At the same time, keeping a large stock is a waste of capital resources, so it should be kept as small as possible. To model that, our goal it to minimize the predicted sum of the stock over the coming days.
{: .notice--success}

Let us start by setting up a typical incorrect model. We assume we are making plans for our restocking for 10 days ahead, and we assume we start from scratch in a situation where we have 2500 items in the inventory (typically you would have some history of sales and restocking orders that you would have to include, and we do this in the final version). To make it even more realistic in terms of what a naive user would do, we write the code in an ugly non-vectorized fashion, and we will also write the dynamical update in a way that hides the simple causality structure. Note that we have to make sure we satisfy the warehouse capabilities both in the morning when new supplies arrive, and in the evening to ensure we aren't running out of stuck when the sales have been done.

````matlab
sdpvar w1 w2 w3 w4 w5 w6 w7 w8 w9 w10
sdpvar s1 s2 s3 s4 s5 s6 s7 s8 s9 s10
sdpvar u1 u2 u3 u4 u5 u6 u7 u8


Model = [uncertain([s1 s2 s3 s4 s5 s6 s7 s8 s9]),
         200 <= [s1 s2 s3 s4 s5 s6 s7 s8 s9] <= 1000,
         w1 == 1200,
         w2 + s1 == w1,
         w3 + s2 == w2 + u1 + 0.1*s1,
         w4 + s3 == w3 + u2 + 0.1*s2,
         w5 + s4 == w4 + u3 + 0.1*s3,
         w6 + s5 == w5 + u4 + 0.1*s4,
         w7 + s6 == w6 + u5 + 0.1*s5,
         w8 + s7 == w7 + u6 + 0.1*s6,
         w9 + s8 == w8 + u7 + 0.1*s7,
        w10 + s9 == w9 + u8 + 0.1*s8,
        0 <= [w1 w2 w3 w4 w5 w6 w7 w8 w9 w10] <= 8000,
        0 <= [w1-s1 w2-s2 w3-s3 w4-s4 w5-s5 w6-s6 w7-s7 w8-s8 w9-s9 w10-s10],        
        0 <= [u1 u2 u3 u4 u5 u6 u7 u8]]
Cost = w1+w2+w3+w4+w5+w6+w7+w8+w9+w10        
````
Let us untangle all the mistakes. First, we have written the dynamical update of the stock in a way that hides the simple update law. It looks like all variables are coupled by a large number of equality constraints, but that's not true. The stock is completely and uniquely goverened by a linear time-invariant dynamical system with states \\(w_i\\) and external inputs \\(u_i\\) and \\(s_i\\). Secondly, the future stock is not a decision variable, it is an expression built from the initial stock, the sequence of sales, and the sequence of restocking order (which is the only decision variable). Hence, a first step would be create the predicted future states using the corresponding assignments

````matlab
sdpvar s1 s2 s3 s4 s5 s6 s7 s8 s9 s10
sdpvar u1 u2 u3 u4 u5 u6 u7 u8

w1 = 2500;
w2 = w1-s1;
w3 = w2-s2+u1+0.1*s1;
w4 = w3-s3+u2+0.1*s2;
w5 = w4-s4+u3+0.1*s3;
w6 = w5-s5+u4+0.1*s4;
w7 = w6-s6+u5+0.1*s5;
w8 = w7-s7+u6+0.1*s6;
w9 = w8-s8+u7+0.1*s7;
w10 = w9-s9+u8+0.1*s8;

Model = [uncertain([s1 s2 s3 s4 s5 s6 s7 s8 s9 s10]),
         200 <= [s1 s2 s3 s4 s5 s6 s7 s8 s9 s10] <= 1000,
           0 <= [w1 w2 w3 w4 w5 w6 w7 w8 w9 w10] <= 8000,          
           0 <= [w1-s1 w2-s2 w3-s3 w4-s4 w5-s5 w6-s6 w7-s7 w8-s8 w9-s9 w10-s10],
           0 <= [u1 u2 u3 u4 u5 u6 u7 u8]]
Cost = w1+w2+w3+w4+w5+w6+w7+w8+w9+w10        
````

This model makes sense physically. Our only decisions are the restocking orders for the coming 10 days, and there are no equalities involving uncertainties, but all future warehouse scenarios are simple maps from realizations of sales, and the restocking plan. Note that **w10** is a complex expression containing all sales variables and all restocking order variables. 

From a planning perspecive, the model is horrible though. It says Alice has to decide all future restocking orders now, before knowing anything about how the sales goes. We are forcing a fixed decision on all future 8 restocking orders already today, and not epxloiting the fact that our purchase order two days ahead should be based on how much we sell today and tomorrow. We will improve that soon, but first let us solve this so called open-loop problem, or non-recourse.

````matlab
optimize(Model,Cost)        
value([u1 u2 u3 u4 u5 u6 u7 u8])

ans =

 400  900  900  900  900  900  900  900
````
If you let the sales be at its maximum, you will see that this fixed plan satisfies the constraints in that the warehouse will be completely emptied every day, but restocked in the morning to be able to handle a worst-case maximum sale scenario once again (900 shipped in for restocking, 100 returned from two days back, and then every sold out again). It will also handle the worst-case that only 200 items are sold everyday, and the warehouse will manage to keep the stock smaller than the upper limit of 8000 items every day. All combinations in between will also be handled, as that is the whole concept of robust optimization.

If we are interested in the worst-case cost, we cannot look at the value of the variable **Cost**, as this expression involves uncertain variables, and these do not have any particular computed value. In the optimization, we have only ensured that the constraints are satisfied for all possible realizations of the uncertainty, and not computed any explicit worst-case realization. In the same sense, all we know is that the worst-case cost has been minimized. To extract this worst-case cost, we simply add a bound on the cost, minimize this bound, and look at the bound instead

````matlab
sdpvar t
optimize([Model,Cost <= t],t)        
value(t)
ans =
   45120

````
In other words, on average over the 10 days, we will have a stock of 4512 in the worst-case (which is when only 200 items are sold so the warehouse inventory builds up to unnecessarily large volumes).

Clearly, the fixed plan is stupid. A first improvement is to change the restocking plan to take known sales into account. If we sell \\(s_i\\) items tomorrow, we can already now say that we should make an order tomorrow evening which compensates for the sales, and then let the optimization procedure adjust this with some fixed plan to optimize the worst-case cost. Hence, we use \\(u_i = v_i + s_i\\) where \\(v_i\\) is our decision variable. 

````matlab
sdpvar s1 s2 s3 s4 s5 s6 s7 s8 s9 s10
sdpvar v1 v2 v3 v4 v5 v6 v7 v8

w1 = 2500;
u1 = v1 + s1;
w2 = w1 - s1;
u2 = v2 + s2;
w3 = w2-s2+u1+0.1*s1;
u3 = v3 + s3;
w4 = w3-s3+u2+0.1*s2;
u4 = v4 + s4;
w5 = w4-s4+u3+0.1*s3;
u5 = v5 + s5;
w6 = w5-s5+u4+0.1*s4;
u6 = v6 + s6;
w7 = w6-s6+u5+0.1*s5;
u7 = v7 + s7;
w8 = w7-s7+u6+0.1*s6;
u8 = v8 + s8;
w9 = w8-s8+u7+0.1*s7;
w10 = w9-s9+u8+0.1*s8;

Model = [uncertain([s1 s2 s3 s4 s5 s6 s7 s8 s9 s10]),
         200 <= [s1 s2 s3 s4 s5 s6 s7 s8 s9 s10] <= 1000,
         0 <= [w1 w2 w3 w4 w5 w6 w7 w8 w9 w10] <= 8000,
         0 <= [w1-s1 w2-s2 w3-s3 w4-s4 w5-s5 w6-s6 w7-s7 w8-s8 w9-s9 w10-s10],
         0 <= [u1 u2 u3 u4 u5 u6 u7 u8]]
Cost = w1+w2+w3+w4+w5+w6+w7+w8+w9+w10;
sdpvar t
optimize([Model,Cost <= t],t)        
value(t)
ans =

   19660
````

This simple rule leads to a massively improved worst-case capital cost. Can we improve it by not just restocking exactly the sold items plus or minus some fixed term, but optimize over this policy?

````matlab
sdpvar s1 s2 s3 s4 s5 s6 s7 s8 s9 s10
sdpvar t1 t2 t3 t4 t5 t6 t7 t8

w1 = 2500;
u1 = v1 + t1*s1;
w2 = w1 - s1;
u2 = v2 + t2*s2;
w3 = w2-s2+u1+0.1*s1;
u3 = v3 + t3*s3;
w4 = w3-s3+u2+0.1*s2;
u4 = v4 + t4*s4;
w5 = w4-s4+u3+0.1*s3;
u5 = v5 + t5*s5;
w6 = w5-s5+u4+0.1*s4;
u6 = v6 + t6*s6;
w7 = w6-s6+u5+0.1*s5;
u7 = v7 + t7*s7;
w8 = w7-s7+u6+0.1*s6;
u8 = v8 + t8*s8;
w9 = w8-s8+u7+0.1*s7;
w10 = w9-s9+u8+0.1*s8;

Model = [uncertain([s1 s2 s3 s4 s5 s6 s7 s8 s9 s10]),
         200 <= [s1 s2 s3 s4 s5 s6 s7 s8 s9 s10] <= 1000,
         0 <= [w1 w2 w3 w4 w5 w6 w7 w8 w9 w10] <= 8000,
         0 <= [w1-s1 w2-s2 w3-s3 w4-s4 w5-s5 w6-s6 w7-s7 w8-s8 w9-s9 w10-s10],
         0 <= [u1 u2 u3 u4 u5 u6 u7 u8]]
Cost = w1+w2+w3+w4+w5+w6+w7+w8+w9+w10;
sdpvar t
optimize([Model,Cost <= t],t)        
value(t)
ans =

   19660
````

No improvement. However, since the dynamics evolve over several days with returned items and deliveries, we should perhaps use more information in our policies. The restocking order next week should be allowed to use all information gathered up until that evening.

````matlab
sdpvar s1 s2 s3 s4 s5 s6 s7 s8 s9 s10
sdpvar v1 v2 v3 v4 v5 v6 v7 v8
sdpvar t1 t2(1,2) t3(1,3) t4(1,4) t5(1,5) t6(1,6) t7(1,7) t8(1,8)

w1 = 2500;
u1 = v1 + t1*s1;
w2 = w1 - s1;
u2 = v2 + t2*[s1 s2]';
w3 = w2-s2+u1+0.1*s1;
u3 = v3 + t3*[s1 s2 s3]';
w4 = w3-s3+u2+0.1*s2;
u4 = v4 + t4*[s1 s2 s3 s4]';
w5 = w4-s4+u3+0.1*s3;
u5 = v5 + t5*[s1 s2 s3 s4 s5]';
w6 = w5-s5+u4+0.1*s4;
u6 = v6 + t6*[s1 s2 s3 s4 s5 s6]';
w7 = w6-s6+u5+0.1*s5;
u7 = v7 + t7*[s1 s2 s3 s4 s5 s6 s7]';
w8 = w7-s7+u6+0.1*s6;
u8 = v8 + t8*[s1 s2 s3 s4 s5 s6 s7 s8]';
w9 = w8-s8+u7+0.1*s7;
w10 = w9-s9+u8+0.1*s8;

Model = [uncertain([s1 s2 s3 s4 s5 s6 s7 s8 s9 s10]),
         200 <= [s1 s2 s3 s4 s5 s6 s7 s8 s9 s10] <= 1000,
         0 <= [w1 w2 w3 w4 w5 w6 w7 w8 w9 w10] <= 8000,
         0 <= [w1-s1 w2-s2 w3-s3 w4-s4 w5-s5 w6-s6 w7-s7 w8-s8 w9-s9 w10-s10],
         0 <= [u1 u2 u3 u4 u5 u6 u7 u8]]
Cost = w1+w2+w3+w4+w5+w6+w7+w8+w9+w10;
sdpvar t
optimize([Model,Cost <= t],t)
value(t)
ans =

   19480
````

Nope. Taking todays sale into account is sufficient in this scenario. At this point though, it shoulds be evident that one should vectorize the code. We do that, but go back to our previous simple policy.

````matlab
s = sdpvar(1,10);
v = sdpvar(1,9);
t = sdpvar(1,9);

w = []
u = [];
for i = 1:10
  if i == 1    
    w = [w 2500];
  elseif i == 2
    w = [w w(i-1)-s(i-1)];
  else
    w = [w w(i-1)-s(i-1)+u(i-2)+0.1*s(i-2)];
  end
  if i <= 9
    u = [u v(i)+s(i)];
  end
end
   
Model = [uncertain(s),
         200 <= s <= 1000,
         0 <= w <= 8000,
         0 <= w-s,
         0 <= u]
Cost = sum(w);
sdpvar t
optimize([Model,Cost <= t],t)
````

We can take this one step further and see the dynamics as a standard discrete-time dynamical system. Let us denote the state \\(x_i\\), and this state contains (\\w_i, s_{i-1} and u_{i-1}\\). With this definition, we have

$$
x_{i+1} = Ax_i + Bu_i + Gs_i = \begin{pmatrix}1 & 0.1 & 1\\
                         0 &   0 & 0\\
                         0 &   0 & 0\end{bmatrix} + 
                         \begin{pmatrix}
                         0\\
                         0\\
                         1\end{bmatrix}u_i+
                         \begin{pmatrix}
                         -1\\
                         1\\
                         0\end{bmatrix}s_i                                                  
$$

With this notation, the prediction model is very compact. Note that we use unnecessarily many \\(u\\), but that's just to keep the code simple. They are defined, but never

````matlab
s = sdpvar(1,10);
v = sdpvar(1,9);
A = [1  0.1  1;
     0    0  0;
     0    0  0]     
B = [0 0 1]';     
G = [-1 1 0]';     

x = [2500;0;0];
u = [];
for i = 1:9
 u = [u s(i) + v(i)];
 x = [x A*x(:,i) + B*u(i) + G*s(i)]; 
end
   
Model = [uncertain(s),
         200 <= s <= 1000,
         0 <= x(1,:) <= 8000,
         0 <= x(1,:)-s,
         0 <= u]
Cost = sum(x(1,:));
sdpvar t
optimize([Model,Cost <= t],t)
````

At this point, one should note that this is a very special case of [robust model predictive control](/example/robustmpc))
