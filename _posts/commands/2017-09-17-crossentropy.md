---
layout: single
category: command
author_profile: false
excerpt: ""
title: crossentropy
tags: [Exponential cone]
comments: true
date: '2016-09-17'
sidebar:
  nav: "commands"
---

Images:mini-category.gif [!Evaluation-based nonlinear operators]

### Description
[crossentropy] is defined as \\(-\sum x_i.*log(y_i)\\). Read more on [Wikipedias article on cross entropty](http://en.wikipedia.org/wiki/Cross_entropy).

### Syntax

````matlab
z = crossentropy(x,y)
````


### Implementation

[crossentropy] is implemented using the [Tutorials.NonlinearOperators evaluation-based nonlinear operators] framework.

### See also

[entropy], [kullbackleibler]
