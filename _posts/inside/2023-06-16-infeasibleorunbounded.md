---
category: inside
subcategory: 2
permalink: infeasibleorunbounded
excerpt: "...or both?"
title: Infeasible or unbounded
tags: [Debugging]
date: 2023-06-16
---

Some solvers use intricate primal-dual presolve routines and reformulations which can cause them to lose track of what a detected inconsistency in the model is caused by, infeasibility or unboundedness. As a result, they simply return the diagnostic *Infeasible or unbounded*.

Untangling this from YALMIP is simple. Unboundedness can only come from the objective, hence remove the objective from the model and try again. If the message changes to *Infeasible* you know the model is infeasible and should start addressing that. If the solver instead solves the problem when the objective was removed, you know the model is unbounded, and should address that problem.

How to analyze a model for infeasibility is explained in the post  [debugging infeasible models](/debugginginfeasible). In short, remove constraints systematically until it becomes infeasible and you know where the peroblem lies, or add slacks to model and see which are activated.

How to analyze a model for unboundedness is explained in the post [debugging unbounded models](/debuggingunbounded). In short, add large bounds on all variables (or small penalities) to make the model bounded, and see which variables become large when that problem is solved.
