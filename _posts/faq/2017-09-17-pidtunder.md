---
layout: single
category: faq
author_profile: false
excerpt: 
title: PID Tuner or linearize in Simulink does not work
tags:
comments: true
date: '2016-09-17'
sidebar:
  nav: "commands"
---

YALMIP has a file called linearize.m which causes a name clash. Most likely you do not need it, so you can delete it (located in the **/extras** directory)
