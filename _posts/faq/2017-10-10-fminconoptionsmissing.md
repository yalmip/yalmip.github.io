---
category: faq
title:  Why are some fmincon options missing?
date: '2017-10-10'
sidebar:
  nav:
---

YALMIP creates the options structure for [FMINCON](/solver/fmincon)  through the call **fmincon('default')** which creates a different set of options compared to the alternative method **optimset('fmincon')**. This is an inconsistency within [FMINCON](/solver/fmincon), and you simply have to figure out what the option is called when created this way.
