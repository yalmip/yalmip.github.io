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

The operator is implemented as [callback operator](/tutorial/nonlinearoperatorscallback), except when [SCS](/solver/scs) or [ECOS](/solver/ecos) are used and the [exponential cone description](/tutorials/exponentialcone) is exploited in a [graph representation](/tutorial/nonlinearoperatorsgraphs).
