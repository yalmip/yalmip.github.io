---
layout: single
category: faq
author_profile: false
excerpt: 
title:  I define a semidefinite constraint, but YALMIP declares it elementwise
tags:
comments: true
date: '2016-09-17'
sidebar:
  nav: "commands"
---

YALMIP detects semidefinite constraints by checking symmetry. Most likely, you have made a mistake and defined a non-symmetric matrix. In some cases (working with very ill-conditioned data), numerical problems may lead to a small violation of symmetry in MATLAB, and YALMIP will declare the constraint as element-wise. To solve this problem, just symmetrize your variable first (X=(X+X')/2).
