---
layout: single
category: example
author_profile: false
excerpt: "Ever wondered how the L1 Chebyshev ball can be computed?"
title: Polytopic geometry using YALMIP and MPT
tags: [Polytopes, MPT, Convex hull, Geometry]
comments: true
date: '2016-09-16'
header:
  teaser: "mptyalmip9.png"
sidebar:
  nav: "examples"
image:
  feature: lofberg.jpg
  teaser: lofberg.jpg
  thumb: lofberg.jpg
---

The toolboxes YALMIP and [MPT](/solver/mpt) were initially developed independently, but have over the years seen more and more integration. Several functionalities in [MPT](/solver/mpt) require YALMIP, and several functionalities in YALMIP require [MPT](/solver/mpt)(/solver/mpt).

In this article, we will look at some examples where we use a combination of YALMIP and [MPT](/solver/mpt) commands, and we will also illustrate how some operations in [MPT](/solver/mpt) can be done using YALMIP code only, and vice versa.

### Plotting polytopes

A cornerstone in [MPT](/solver/mpt) is the library for definition and manipulation of polytopes. Let us define a random polytope, and plot it using [MPT](/solver/mpt)(/solver/mpt).

````matlab
A = randn(10,2);
b = 3*rand(10,1);
P = polytope(A,b);
plot(P);
````

![costs_LPVMPC]({{ site.url }}/images/mptyalmip1.png){: .center-image }

The same plot can be achived by pure YALMIP code

````matlab
x = sdpvar(2,1);
plot(A*x <= b);
````

Alternatively, we can use a combination of YALMIP and [MPT](/solver/mpt)(/solver/mpt). The result is the same YALMIP model as above.

````matlab
plot(ismember(x,P));
````

A final approach is to define the problem using YALMIP code, but convert it to a polytopic object

````matlab
plot(polytope(A*x <= b));
````

Note that the algorithm to plot the sets are completely different in YALMIP and [MPT](/solver/mpt)(/solver/mpt). [MPT](/solver/mpt) explicitly works with the polytopic structure and computes all vertices using a vertex enumeration algorithm. YALMIP on the other hand makes no assumption about the sets. To be able to handle arbitrary sets, it employs a ray-shooting strategy to generate points on the boundary of the feasible set, and then plot this (inner) approximation.

### Minkowski difference

Minkowski difference, or polytope substraction, is overloaded on polytope objects in [MPT](/solver/mpt)(/solver/mpt). We define a new polytope, and subtract it from our first polytope

````matlab
E = randn(10,2);
f = 0.1*rand(10,1);
S = polytope(E,f);
plot(P-S,'y');
````

![costs_LPVMPC]({{ site.url }}/images/mptyalmip2.png){: .center-image }

To accomplish the same using only YALMIP, we have to go back to the defintion of the Minkowski difference. The Minkowski difference \\(P-S\\) is defined as all \\(x\\) such that \\(x+w\\) is in \\(P\\) for all \\(w\\) in \\(S\\). This can be interpreted as a robust optimization problem.

````matlab
w = sdpvar(2,1);
plot([A*(x+w) <= b, E*w <= f, uncertain(w)]);
````
Once again, we implement this using a combination of YALMIP and [MPT](/solver/mpt) commands
````matlab
plot([ismember(x+w,P), ismember(w,S), uncertain(w)]);
````

Note that the use of the [robust optimization framework] in YALMIP allows us to generalize the Minkowski difference to subtract an arbitrary conic set. [MPT](/solver/mpt) on the other hand is limited to polytopic sets.

````matlab
w = sdpvar(2,1);
plot([A*(x+w) <= b, w'*w <= 0.08^2, uncertain(w)]);
````

![costs_LPVMPC]({{ site.url }}/images/mptyalmip3.png){: .center-image }

### Minkowski sum

Addition of polytopes, i.e. Minkowski sum, is also directly available in [MPT](/solver/mpt)

````matlab
plot(P+S)
````

![costs_LPVMPC]({{ site.url }}/images/mptyalmip4.png){: .center-image }

Once again, to plot this using YALMIP, we use the definition of Minkowski sum, the set of all \\(z\\) such that \\(z=x+w\\) where \\(x\\) is in \\(P\\) and \\(w\\) is in \\(S\\). We thus create a new variable \\(z\\), use the definition, and plot the feasible set in the \\(z\\)-space.

````matlab
z = sdpvar(2,1);
plot([z == x+w, A*x <= b, E*w <= f],z)
````

### Chebychev balls

The [Chebyshev ball](http://en.wikipedia.org/wiki/Chebyshev_center) is defined as the largest possible ball that can be placed inside a set \\(P\\). The search for the Chebychev center, and the associated size of the ball, can be stated as finding the largest \\(r\\) such that \\(x+rd\\) is in \\(P\\) for all \\(d\\) in a unit-ball.

For a polytope and the Euclidean norm-ball, this boils down to

````matlab
xc = sdpvar(2,1);
r = sdpvar(1);
optimize(A*xc+r*sqrt(sum(A.^2,2)) <= b,-r)
````

We plot the polytope and the Chebychev ball

````matlab
plot(A*x < b);hold on
plot(norm(x-value(xc),2)<=value(r),x,'b')
````

![costs_LPVMPC]({{ site.url }}/images/mptyalmip8.png){: .center-image }

In order to extend this concept, we first note that the Chebychev formulation can be interpreted as a robustness problem. Hence, we can compute the Chebychev ball using the robust optimization module

````matlab
d = sdpvar(2,1);
optimize([A*(xc+r*d) <= b, uncertain(d), d'*d <= 1],-r)
````

With this formulation at hand, we can generalize. The robust optimization module allows fairly general uncertainty descriptions when the uncertain constraint is elementwise. Let's compute the 1-norm and infinity-norm Chebychev balls!

````matlab
plot(A*x < b);
optimize([A*(xc+r*d) <= b, d'*d < 1, uncertain(d)],-r)
hold on
plot(norm(x-value(xc))<=value(r),x,'b')

optimize([A*(xc+r*d) <= b, norm(d,1)<=1, uncertain(d)],-r)
plot(norm(x-value(xc),1)<=value(r),x,'g')

solvesdp([A*(xc+r*d) <= b, norm(d,inf) <= 1, uncertain(d)],-r)
plot((abs(x-value(xc)))<=value(r),x,'y')
````

![costs_LPVMPC]({{ site.url }}/images/mptyalmip9.png){: .center-image }

### Projection

Define a polytope in 3D, and project it two its two first coordinates.

````matlab
H = randn(10,3);
k = rand(10,1);
R = polytope(H,k);
plot(R);hold on
Q = projection(R,1:2);
plot(Q);
````

![costs_LPVMPC]({{ site.url }}/images/mptyalmip5.png){: .center-image }

The same figure can be generated using YALMIP code

````matlab
x = sdpvar(3,1);
plot(H*x <= k);hold on
plot(H*x <= k,x(1:2),'k');
````

Crucial to understand is that YALMIP does not explicitly generate any projection but simply plots the projection of the results from the ray-shooting.

However, if we want the explicit projection, the command projection is available on polytopic constraints. YALMIP will convert the constraints to an [MPT](/solver/mpt) polytope object, apply the projection, and the reconstruct a YALMIP constraint.

````matlab
R = [H*x <= k]
Q = projection(R,x(1:2))
plot(R);hold on;
plot(Q,'k');
````


### Slicing

[MPT](/solver/mpt) has support for slicing. This command allows you to fix the value of a set of variables, and plot the polytope in the remaining free variables. Let us fix the first coordinate to the value 0.2 in the 3D polytope above

````matlab
plot(R);hold on;
plot(slice(R,1,0.2))
````

![costs_LPVMPC]({{ site.url }}/images/mptyalmip6.png){: .center-image }

The same thing can be accomplished with the following YALMIP code. Note that we have to specify that we want to plot the remaning 2D polytope. Otherwise, YALMIP will try to plot a flat polytope in 3D

````matlab
plot([H*x <= k, x(1)==0.2],x(2:3))
````

Alternatively, we can eliminate x(1) from the problem, and plot the 2D polytope that remains.

````matlab
plot(replace(H*x,x(1),0.2) <= k)
````

### Convex hull

Both [MPT](/solver/mpt) and YALMIP can be used to obtain the convex hull of the union of polytopes. Using [MPT](/solver/mpt)(/solver/mpt), we quickly define two cubes and plot them and their [convex hull](/commands/hull)

````matlab
P1 = unitbox(2,0.5)+[1;1];
P2 = unitbox(2,0.5)+[-1;-1];
plot([hull([P1 P2]) P1 P2])
````

![costs_LPVMPC]({{ site.url }}/images/mptyalmip7.png){: .center-image }

A pure YALMIP version would be

````matlab
x = sdpvar(2,1);
P1 = -0.5 <= x-[1;1] <= 0.5;
P2 = -0.5 <= x+[1;1] <= 0.5;
P  = hull(P1,P2);
plot(P,x);hold on
plot(P1,x,'y');
plot(P2,x,'b');
````

Once again, note that [MPT](/solver/mpt) and YALMIP use different approaches to construct the convex hull. [MPT](/solver/mpt)  is based on a vertex enumeration of the individual polytopes. YALMIP on the other hand is based on a general lifting approach involving additional variables and constraints (this is the reason we explicitly tell YALMIP to plot the convex hull w.r.t. the original variables, since auxiliary variables have been introduced). The benefit of this approach is of course that the method applies to arbitrary conic-representable sets as described in the [convex hull command](/commands/hull)
