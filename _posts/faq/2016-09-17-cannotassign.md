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


This is because of the way the object orientation works in MATLAB. The only work-around now is the command **X=double2sdpvar(eye(2));X(1,1) = sdpvar(1,1);** (or more sensibly manually construct the matrix by direct concatenations)
