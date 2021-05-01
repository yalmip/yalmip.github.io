---
layout: single
excerpt: "There is more than one way to skin a cat"
title: Nonconvex long-short constraints - 7 ways to count
tags: [Integer programming, Logic programming, Cardinality, Finance, Portfolio optimization]
comments: true
date: 2017-06-27
---

A question on the forum today asked how one can constrain a solution to ensure a certain percentage of a vector to have a particular sign. This could for instance be of relevance in a [portfolio allocation](/example/portfolio) problem where we have constraints on the ratio of long (positive) and short (negative) positions.

There are many natural ways to write this in MATLAB, but to include it in YALMIP code, it has to be based on operators which are supported by YALMIP. [Let's look at some alternatives](/examples/longshort)

