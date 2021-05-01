---
category: faq
title: YALMIP complains about failing in convexity propagation
tags: [Epigraph, Hypograph, Sqrt, Norm]
date: '2016-09-17'
sidebar:
  nav:
---

This means that you have used so called nonlinear operators to model your problem, and most likely you have defined a problem which cannot be represented using standard convex constraints. If you know that your model is convex, try to model the nonlinear terms by hand to see if you actually are correct (the convexity analysis is conservative, although in most cases failure in the convexity propagation is due to actual nonconvexity.
