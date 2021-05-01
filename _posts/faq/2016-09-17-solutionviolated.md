---
category: faq
title: The solution I get is not feasible but violated by, say, -1e-6
date: '2016-09-17'
sidebar:
  nav:
---

Most solvers actually use infeasible/exterior algorithms, so slightly infeasible solutions are common. If you really need a feasible point (assuming one really exists), simply define a slightly tighter constraint than wanted (e.g., change your constraint from **X>=0** to **X>=eye(n)*smallconstant** in the case of semidefinite constraints etc.). 

Note that what you typically are doing then is that you somehow think you can enforce that your car satisfies that it is at least 4 meters long, and not 4 meters minus the width of an atom....In other words, in almost all practical situations the model you are using has much worse precision than the solver anyway, so trying to trick the solver like this is just silly for all practical means.
