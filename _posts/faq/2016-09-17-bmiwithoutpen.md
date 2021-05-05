---
category: faq
title:  Can I solve BMIs without PENBMI or PENLAB?
tags: [Bilinear matrix inequality, Semidefinite programming]
date: '2016-09-17'
sidebar:
  nav:
---

Yes (and you will have to as these solvers typically do not work well). 

Develop your own heuristic (fix variables, alternating coordinates, trust regions etc). This can easily be done in high-level YALMIP code.

Alternatively, try the global solver [BMIBNB](/solver/bmibnb) which supports nonlinear semidefinite constraints. Most likely it worn't work as your problem is too large, but it never hurts to try.
