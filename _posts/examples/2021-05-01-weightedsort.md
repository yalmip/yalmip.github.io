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

A discussion on stackexchange led to some experiments and an interesting case where a linear programming formulation was absolutely horrible compared to more naive approach using a eneral nonlinear solver.

### Sorting

To begin with, we note that sorting a vector in general is a very complex operation to describe using optimization. Assuming a descending sort, we can describe the sorted vector \\(s = \text{sort}(x)\\) with the constraints \\(s = Px, s_i \geq s_{i+1} \\) where \\(P\\) is a binary permuation matrix \\( \sum_i P_{ij} = 1, \sum_j P_{ij}=1\\). To arrive at a MILP representation, a linearization (i.e. big-M representation) of the product \\(Px\\) is needed. Hence, the full model will not only introduce the large binary matrix \\(P\\) but also a large amount of additonal variables and constraints to represent the product. Nevertheless, this is the model you obtain if you use the readily available command [sort](/command/sort).

