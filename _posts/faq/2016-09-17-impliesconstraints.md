---
category: faq
title: Using implies on constraint sometimes fails
date: '2016-09-17'
sidebar:
  nav:
---

Using the logical constraint [implies](/command/implies) with the first argument being a constraint can easily lead to numerically sensitive models, and should thus be avoided. You should partition your logical model to a set of cases and then use a binary variable for each case, implying a set of constraints.
