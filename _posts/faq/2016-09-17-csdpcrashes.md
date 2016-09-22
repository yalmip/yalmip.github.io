---
layout: single
category: faq
author_profile: false
excerpt: 
title: CSDP runs but crashes
tags:
comments: true
date: '2016-09-17'
sidebar:
  nav:
---

Running MATLAB 6.1 and CSDP 4.6? In that case, edit readsol.m in the CSDP directory and replace all occurrences of && with &. Even better, download the latest version of CSDP.
