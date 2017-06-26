---
layout: single
category: faq
author_profile: false
excerpt: 
title: I can not write X = eye(2); X(1,1)=sdpvar(1,1)!
tags:
comments: true
date: '2016-09-17'
sidebar:
  nav:
---


This is because of the way the object orientation works in MATLAB (a user-class object cannot be assigned into a built-in class object). The prefered pattern to code in YALMIP is to use concatenation, i.e. create it as  **X=[sdpvar(1,1) 0;0 1]**. 

A possible alternative is the command **X=double2sdpvar(eye(2));X(1,1) = sdpvar(1,1);** (not recommended though as it misses the core philosophy of YALMIP of being completely natural to beginners)
