---
layout: single
category: command
author_profile: false
excerpt: "Convexity aware implementation of kullackleibler entropy"
title: kullbackleibler
tags: [Exponential cone programming representable, Exponential and logarithmic functions]
comments: true
date: '2016-09-17'
sidebar:
  nav: "commands"
---

[kullbackleibler](/command/kullbackleibler) is defined as \\( \sum x_i \log(x_i/y_i) \\). Read more on [Wikipedias article on Kullback-Leibler divergence](http://en.wikipedia.org/wiki/Kullback%E2%80%93Leibler_divergence).

### Syntax
````matlab
z = kullbackleibler(x,y)
````

### Comments

The operator  is implemented as a [callback operator](/tutorial/nonlinearoperatorscallback), except when [Mosek 9](/solver/mosek), [SCS](/solver/scs) or [ECOS](/solver/ecos) are used and the [exponential cone](/tutorial/exponentialcone) description is exploited in a [graph representation](/tutorial/nonlinearoperatorsgraphs).
