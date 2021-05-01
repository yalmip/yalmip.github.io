---
category: faq
title:  I define a semidefinite constraint, but YALMIP declares it elementwise
date: '2016-09-17'
sidebar:
  nav:
---

YALMIP detects semidefinite constraints by checking symmetry as explained in the [basic introduction](/tutorial/basics). Most likely, you have made a mistake and defined a non-symmetric matrix. 

In some cases (working with very ill-conditioned data), numerical problems may lead to a small violation of symmetry in MATLAB, and YALMIP will declare the constraint as element-wise. To solve this problem, just symmetrize your variable (**X=(X+X')/2**).
