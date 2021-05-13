---
category: inside
subcategory: 2
excerpt: "Wanted but not needed"
title: Strict inequalities
tags: [Common mistakes]
date: 2021-05-13
---

It finally had to be done. Strict inequalities which were avaiable in the language initially but have raised increasingly more annoying warnings over the last decade, are no longer possible to use.

Note that being available never has meant that strict inequalities were applied in any solver, it just meant they were (increasingly less) silently changed to non-strict inequalities.

## Why strict inequalities make no sense - theory

If you think strict inequalities are relevant, then write down the solution to the following problem.

$$
\begin{aligned}
\text{minimize } & x\\
\text{subject to } & x > 0
\end{align}
$$

There is no minimizer to this problem due to the open feasible set coming from the strict inequality. No matter what solution I, you, or some solver returns, we can always complain and say that it is sub-optimal.

If you think in floating-point numbers you might be cheeky and say that the solution in MATLAB should be '2.2251e-308' which is the smallest real number MATLAB can generate. But then you no longer solved the strict problem but solved the problem with \\( x\geq 2.2251\cdot 10^{-308}\\) which makes no sense to state in practice as discussed below.

You might complain and say that you do not require the optimal solution, but only a good enough solution. Then we have to ask why you bother with the strict inequality to begin with. Simply replace it with some strict inequality bounded away from zero. You good-enough-tolerance is just another way of expressing a margin to non-strict.

## Why strict inequalities make no sense - practice

If you somehow managed to tell the solver to only return strict solution, this epsilon-strictness would drown in the general tolerances and margins used in a solver. Already with a non-strict constraints, solvers typically only promise that they returns solutions which are numerically close to feasible, using tolerances of, say, \\(10^{-8}$.

Essentially all numerical solvers interfaced in YALMIP work with infeasible methods which approach the optimal solution not necessarily from the feasible region. The term interor-point might fool some to think that the solver definitely works in the interior, but this interior is not necessarily the interior of your model, but can be interior of some lifted/slacked/dual space.

## What to do then?

So you really want a strict solution, but you only have strict constraints to work with. The first thing you should do is to really confirm that you need strict solutions. If you still need this, a simple approah is to simply skip this in the modelling phase, and just check that the solution satisfies your strictness condition. If this does not hold, you will have to force the solver to stay away from your forbidden region. What this means is that you have to add margins in your constraints, scalar or semidefinite depending on the set you are working with.

````matlab
X >= my_magic_margin*eye(n)
````

This is where it becomes tricky. First you have to remember that solver have their tolerances, so if you use a magic margin of \\(10^{-15}\\) it will problably make absolutely no difference compared to leaving it out, as it drowns in the tolerances used for declaring feasibility by the solver anyway.

On the other hand, if you use a large margin to be on the safe side, you might reduce the feasible set considerably or even render the problem infeasible.

A (maybe not so) clever reformulation might be to introduce a new variable \\(t\\) and replace \\(x>0\\) with \\(x \geq e^{-t}\\). Not only do you introduce the risk of numerical issues as the solver possibly sends \\(t\\) to infinity, you still have no idea if the solver approaches this constraints with infeasible methods.

## But it is an integer variable!

Writing \\x > 0\\) for an integer variable \\x\\) is just a very bad way of saying \\(x \geq 1\\).
