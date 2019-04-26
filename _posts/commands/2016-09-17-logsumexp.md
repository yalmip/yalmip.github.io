---
layout: single
category: command
author_profile: false
excerpt: ""
title: logsumexp
tags: [Exponential cone programming representable, Exponential and logarithmic functions, Logistic regression]
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

### Comment

The convex [logsumexp](/commands/logsumexp) operator can be used to perform logistic regression, as illustrated in the example in the [exponential cone tutorial](/tutorial/exponentialcone)


### Implementation

The operator is implemented as [callback operator](/tutorial/nonlinearoperatorscallback), except when [Mosek 9](/solver/mosek), [SCS](/solver/scs) or [ECOS](/solver/ecos) are used and the [exponential cone description](/tutorial/exponentialcone) is exploited in a [graph representation](/tutorial/nonlinearoperatorsgraphs).
