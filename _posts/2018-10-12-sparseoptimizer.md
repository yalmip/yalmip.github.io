---
layout: single
permalink: /sparseoptimizer
excerpt: "Be careful with unnecessary symbolic overhead"
title: "Working with sparse parameterizations in optimizer"
header:
  teaser: sparseA.png
date: 2018-10-12
---

The [optimizer framework](/command/optimizer) can be used to reduce overhead significantly when solving many similiar problems, by pre-compiling a model parameterized in set of parameters which can change.

Unfortunately, in some cases the precompiled approach can become slower than a standard modelling approach. The mistake is typically a parameterization which does not exploit sparsity. [Learn how to avoid this...](/example/sparseoptimizer)
