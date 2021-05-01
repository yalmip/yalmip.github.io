---
layout: single
category: example
author_profile: false
excerpt: "Doing the forbidden"
title: Sorting with linear programming
tags: [Sort, linear programming, black box]
comments: true
date: '2021-05-01'
published: false
header:
  teaser: "starshaped2.png"
sidebar:
  nav: "examples"
image:
  feature: lofberg.jpg
  teaser: lofberg.jpg
  thumb: lofberg.jpg
---

A discussion on stackexchange led to some experiments and an interesting case where a linear programming formulation is absolutely horrible compared to an approach using a general nonlinear solver. We thus have a problem where the basic idea that we should solve as a linear program if we can formulate it as such is violated.


## The problem

The problem discussed which triggered this example is

$$
\begin{aligned}
\text{minimize}_w ~~& \sort{Rx}^Tp + ||w-w_0||_1 \\
\text{subject to} ~~& ||w||_{\infty}  leq 1
\end{aligned}
$$

The norm in the objectiv and the norm in the constraints are trivially LP-representable and do not pose any problems, neither in theory nor practice. The problem is the first term which involves a sorting operator. Note though that it does not explicitly require the sorted vector, it only asks for the inner products of the sorted vector and a given vector \(p\\). In our discussion here, we assume the sort is in descending order.

For arbitrary \\(p\\) this is not a convex problem. This can easily be seen if we consider the a case where all elements in \\(p\\) are 0, and the last element is 1. The inner product then simply returns the smallest element of the vector \\( Rw\\), and we know that \\( \min \\) is a concave function, hence the problem is non-convex as we minimize.

### MILP sorting

To begin with, we note that sorting a vector in general is a very complex operation to describe in an optimization model. Assuming a descending sort, we can describe a sorted vector \\(s = \text{sort}(x)\\) with the constraints \\(s = Px, s_i \geq s_{i+1} \\) where \\(P\\) is a binary permuation matrix, \\( \sum_i P_{ij} = 1, \sum_j P_{ij}=1\\). To arrive at a MILP representation, a linearization (i.e. big-M representation) of the product \\(Px\\) is needed. Hence, the full model will not only introduce the large binary matrix \\(P\\) but also a large amount of additonal variables and constraints to represent the linearized product. This is the model you obtain if you use the readily available command [sort](/command/sort).

In the general case, the problem is nonconvex and we have no hope of an LP representation and thus start with the MILP formulation. We can test on a small random case.

````matlab
m = 4;
n = 10;
p = randn(n,1);
R = randn(n,m);
w0 = randn(m,1);

objective = sort(R*w,'descend')'*p+norm(w-w0,1);
Model = [norm(w,inf)<=1];    
sol = optimize(Model,objective);
````

Increasing the problem size will quickly lead to problems where the solution time starts to become very problematic.





