---
layout: single
category: example
author_profile: false
excerpt: "Sort of convex but not quite"
title: Working with starshaped polygons
tags: [Geometry, interp1, sos2, Special ordered set]
comments: true
date: '2021-04-28'
published: true
header:
  teaser: "mptyalmip9.png"
sidebar:
  nav: "examples"
image:
  feature: lofberg.jpg
  teaser: lofberg.jpg
  thumb: lofberg.jpg
---

** UNDER CONSTRUCTION**

In the world between convex and non-convex sets, there is a geometry which is called [star-shaped (star-domain, star-convex, radially convex)](https://en.wikipedia.org/wiki/Star_domain). As the name reveals, a classical drawing of a star (centered around the origin) is a special case. 

Mathematically, the definition of a star-shaped set is that all points between the origin and any point in the set is in the set. Compare this to the definition of a convex set where all points on a line between any two points are in the set. Loosely speaking, it is convex w.r.t a particular point (here the origin). The origin can be changed to some other point by translating the whole set and defining star convexity w.r.t the translated origin.

Here, we will play around a bit with star convex polygons, modelling them both manually and by using built-in support.

Note that you need a [sos2](/command/sos2) capable solver such as [GUROBI](/solver/gurobi) or [CPLEX](/solver/cplex).

### Star-convex polygons

Define some data represeting the vertices of a star (remember, a star-convex does not have to look like a star, it is just a name)

````matlab
n = 5;
th = linspace(0, 2*pi, 2*n+1);
xi = (1+rem((1:2*n+1),2)).*cos(th);
yi = (1+rem((1:2*n+1),2)).*sin(th);
clf;
hold on;
plot( xi, yi, 'b*-' );
axis equal;
````

![A star]({{ site.url }}/images/starshaped1.png){: .center-image }

The feasible set inside the star can be represented as the union of 6 polytopes. Hence, a general approach to representing this set is to descibe these polytopes, and then use logic programming and binary variables to represent the union as illustrated in [the big-M tutorial](/tutorial/bigmandconvexhulls/). However, star-shaped polygons can also be represented much more conveniently using [sos2](/command/sos2) constructs.

To derive a [sos2](/command/sos2) representation, we first focus on representing the border of the polygon. Every point on the border of the star can be written as a linear combination of two adjacent vertices \\(\lambda_i v_i + \lambda_{i+1}v_{i+1}, \lamba_i + \lambda_{i+1}==1, \lamba_i\geq 0,  \lamba_{i+1}\geq 0\\). This is the classical application of [sos2](/command/sos2), and precisely the same model YALMIP uses for general [interp1](/command/interp1) representation.

Assuming we work with coordinates \\(x\\) and \\(y\\) as decision variables in our model, and we want to say that \\((x,y)\\) is on the border, we arrive at the model

````matlab
sdpvar x y
lambda = sdpvar(length(xi),1)
F = [sos2(lambda), lambda>=0, sum(lambda)==1,
     x == lambda'*xi(:), y == lambda'*yi(:)];
````

That's all!

To see that this really models what we want, we could plot the set. When doing so, you will note that it is not drawing the star, but the convex hull of the star. 

````matlab
plot(Model,[x;y],[],[],sdpsettings('plot.shade',.1) )    
````

![Star convex hull]({{ site.url }}/images/starshaped2.png){: .center-image }

This is not due to an incorrect model, but simply a limitation of the plot command. When you plot a set, it performs ray-shooting to find points on the boundary, and then draws the convex hull of these points (the only way to draw non-convex sets in YALMIP is to explicitly model mixed-integer models, as YALMIP then will perform enumeration of all combinatorial combinations and draw each set individually to depict the union)

To make sure we actually model the border of the polygon, let us solve a simple problem where we find the point closest to a point outside.

````matlab
optimize(Model, (x-1)^2 + (y-1.5)^2)
plot(1,1.5,'o*r')
plot(value(x),value(y),'o*r')
````


So how can we includethe interior? This is where star-convexity comes into play. Since any scaled point on the border also in part of the star, it means we can scale the interpolating \\(\lambda\\) with an arbitrary scale $\leq t = \leq 1$. It also means we can take the adjacent interpolated vertices and scale them individually first, and then interpolate between them. Effectively, this simply means we can replace the model with

````matlab
sdpvar x y
lambda = sdpvar(length(xi),1)
F = [sos2(lambda), lambda>=0,sum(lambda)<=1,
     x == lambda'*xi(:), y == lambda'*yi(:)]
````

What about the translations? Let us start by defining a star centered outside the origin, so that star-convexity w.r.t the origin is violated.

````matlab
n = 5;
th = linspace(0, 2*pi, 2*n+1);
xi = (1+rem((1:2*n+1),2)).*cos(th);
yi = (1+rem((1:2*n+1),2)).*sin(th);
clf;
hold on;
plot( xi, yi, 'b*-' );
grid on
axis equal;
````

If we define the set using the same code as before, the case which only includes the border will still be valid, but the generalization to include the interior will be flawed. As the interpolating \\(\lambda\\) is allowed to be zero, the origin will be included as a feasible point. The problem is that the set is not star-convex w.r.t the origin so the set one will create now is the union of all scaled stars, which is something completely different when star-convexity does not hold. 

No problems though, we shift the origin and define the set as a translated star-convex set. Draw its convex hull just for sanity check.

````matlab
xc = mean(xi);
yc = mean(yi);
lambda = sdpvar(length(xi),1)
Model = [sos2(lambda), lambda>=0,sum(lambda)<=1,
         x == xc + lambda'*(xi(:)-xc), 
         y == yc + lambda'*(yi(:)-yc)];
plot(Model,[x;y],[],[],sdpsettings('plot.shade',.1)     
````

Not too complicated to code, but YALMIP has built-in support for creating these sets even more convenently. By default it only takes the coordinates and represents the star-convex set with an automatic translation of the origin, and there is a third option to only include he border and a fourth argument to translate the whole set (can be a decision variable).

````matlab
Model = starpolygon(xi,yi);
Model = starpolygon(xi,yi,1);        % Only include border
Model = starpolygon(xi,yi,[],[7;5]); % Additional translation
````

