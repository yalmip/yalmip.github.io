---
layout: single
category: faq
author_profile: false
excerpt: 
title: I get strange results when I use the option 'removeequalities
tags:
comments: true
date: '2016-09-17'
sidebar:
  nav: "commands"
---

When equality constraints are removed by YALMIP by deriving a reduced basis ('removeequalities' set to 1 or 2) dual variables will not be recovered. This may lead to further complications in some cases. If you are solving a problem that you have derived by using the function dualize, you original variables will not be computed, since they are computed from the missing duals. Another case is when you solve a linearly parameterized sum of squares problem using a kernel model ('sos.model' set to 0 or 1). The parameterization variables are computed from the dual variables, hence failure will occur. To summarize, do not use the option 'removeequalities' in sum-of-squares problems or dualized problems, unless you really now what you are doing.
