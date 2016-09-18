---
layout: single
category: faq
author_profile: false
excerpt: 
title: fmincon crashes
tags:
comments: true
date: '2016-09-17'
sidebar:
  nav: "commands"
---

Do you have [MOSEK] installed? This can cause problems due to an inconsistency between MATLABs and Moseks implementation of the file optimset.m. Remove [MOSEK] from your path.
