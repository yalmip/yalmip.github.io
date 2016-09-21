---
layout: single
category: command
author_profile: false
excerpt: "Convexity aware implementation of kullackleibler entropy"
title: kullackleibler
tags: [Exponential cone programming representable, Exponential and logarithmic functions]
comments: true
date: '2016-09-17'
sidebar:
  nav: "commands"
---

[[kullbackleibler]] is defined as \\( \sum x_i \log(x_i/y_i) \\). Read more on [Wikipedias article on Kullback-Leibler divergence](http://en.wikipedia.org/wiki/Kullback%E2%80%93Leibler_divergence).

### Syntax
````matlab
z = kullbackleibler(x,y)
````

### Comments

[kullbackleibler] is implemented using the [evaluation-based nonlinear operators] framework, unless [SCS] or [ECOS] is used and the exponential cone property is used.
