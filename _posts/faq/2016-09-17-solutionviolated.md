---
layout: single
category: faq
author_profile: false
excerpt: 
title: The solution I get is not feasible but violated by, say, -1e-6
tags:
comments: true
date: '2016-09-17'
sidebar:
  nav:
---

Most solvers actually use infeasible/exterior algorithms, so slightly infeasible solutions are common. If you really need a strictly feasible point (assuming one really exists), simply define a slightly tighter constraint than wanted (e.g., change your constraint from **X>=0** to **X>=eye(n)*smallconstant** in the case of semidefinite constraints etc.)
