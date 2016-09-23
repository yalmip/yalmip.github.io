---
layout: single
category: command
author_profile: false
excerpt: ""
title: logsumexp
tags: [Exponential cone programming representable, Exponential and logarithmic functions]
comments: true
date: '2016-09-17'
sidebar:
  nav: "commands"
---

[logsumexp](/command/logsumexp) is a convexity aware implementation of \\(\log(\sum_i e^{x_i})\\)

### Syntax

````matlab
y = logsumexp(x)
````

### Implementation

The convex operator [logsumexp](/command/logsumexp) is implemented using the [evaluation-based nonlinear operators] framework unless [SCS](/solver/scs) or [ECOS](/solver/ecos)  is used and an exponential cone representation is used.
