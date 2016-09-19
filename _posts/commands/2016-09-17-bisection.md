---
layout: single
category: command
author_profile: false
excerpt: "Built-in meta-solver for some quasi-convex optimization problems"
title: solvemoment
tags: [Semidefinite programming, Quasi-convex]
comments: true
date: '2016-09-17'
sidebar:
  nav: "commands"
---

[bisection] implements a bisection algorithm to solve some simple quasi-convex problems.

Assumed constraints quasi-convex in \\(x\\) and the scalar \\(t\\), and objective function \\(t\\) or \\(-t\\)
### Syntax

````matlab
diagnostics = bisection(F,t,options)
````

See [the decay-rate example](/examples/decayrate) 
