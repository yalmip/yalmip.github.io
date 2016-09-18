---
layout: single
category: command
author_profile: false
excerpt: ""
title: logsumexp
tags:
comments: true
date: '2016-09-17'
sidebar:
  nav: "commands"
---

Images:mini-category.gif  [!Evaluation-based nonlinear operators]

### Syntax

````matlab
y = logsumexp(x)
````

### Implementation

The convex operator [logsumexp], defined as '''log(sum(exp(x)))''' is implemented using the [Tutorials.NonlinearOperators evaluation-based nonlinear operators] framework. except for when [Solvers.SCS scs] or [Solvers.ECOS ecos]  is used where the exponential cone is used.

### See also
[entropy], [crossentropy], [kullbackleibler]
