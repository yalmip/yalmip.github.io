---
category: faq
title: PENBMI does not work with YALMIP anymore
date: '2016-09-17'
sidebar:
  nav:
---

Version 1.1 and earlier will not work directly anymore. However, this is easily fixed. Edit the file callpenbmim.m (if you use the PENOPT version) or callpenbmi.m (if you use the TOMLAB version). Uncomment the code below the comment "UNCOMMENT THIS".
