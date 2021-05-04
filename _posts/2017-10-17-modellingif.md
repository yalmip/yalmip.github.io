---
layout: single
permalink: /modellingif
excerpt: "Untangle that messy expression"
title: "Modelling if-else-end statements"
comments: true
date: 2017-10-17
---

YALMIP supports complex models by overloading most standard operators in MATLAB. One common issue though that many users struggle with is models involving **if** statements. The object-oriented overloading of operators in MATLAB does not support overloading of programming constructs, i.e., **if x**, **elseif x**, **switch x case y**, **for x=y** where **x** or **y** involves [sdpvar](/command/sdpvar) variables will not work. [Learn how to model if-statements](/examples/modellingif).

