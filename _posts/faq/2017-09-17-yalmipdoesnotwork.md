---
layout: single
category: faq
author_profile: false
excerpt: 
title: YALMIP does not work
tags:
comments: true
date: '2016-09-17'
sidebar:
  nav: "commands"
---

1. Restart MATLAB.
2 Are you running the absolutely latest version of YALMIP. If not, install it.
3. Remove any old version of YALMIP before you install the new version. Do not just copy the new version into the old YALMIP directory.
4. Added all the paths according to [[installation]](tutorial/installation). If you don't know what a path in MATLAB is or how to set it, learn about that first.
5. Are you sure you added all the paths!
6. Could be related to toolbox path caching (open menu File/Preferences/General and run "Update Toolbox Path Cache")
7. Removed your old YALMIP version from the MATLAB path?
8. Added all the paths to your solver?
9. Do you have any solver installed?
10. Compiled the solver (if needed)? Compiled it for the correct MATLAB version?
11. Using MATLAB 5.3.1 or later?
12. Using advanced features such as nonlinear operators (exp, log, sin,...)? If so, you need MATLAB 7.0 or later
13. Probably a PICNIC problem ;-P. Contact me for support.
