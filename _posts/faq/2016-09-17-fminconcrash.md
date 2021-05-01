---
category: faq
title: fmincon crashes
tags: [FMINCON]
date: '2016-09-17'
sidebar:
  nav:
---

Do you have [MOSEK](/solver/mosek) installed? This can cause problems due to an inconsistency between MATLABs and Moseks implementation of the file optimset.m. Remove [MOSEK](/solver/mosek) from your path.
