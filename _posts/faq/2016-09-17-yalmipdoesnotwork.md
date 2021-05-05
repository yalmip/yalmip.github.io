---
category: faq
title: YALMIP does not work
date: '2016-09-17'
sidebar:
  nav:
---

1. Restart MATLAB.
2. Are you running the absolutely latest version of YALMIP. If not, install it.
3. Remove any old version of YALMIP before you install the new version. Do not just copy the new version into the old YALMIP directory.
4. Added all the paths according to [[installation]](/tutorial/installation)? If you don't know what a path in MATLAB is or how to set it, learn about that first.
5. Are you sure you added all the paths!
6. Could be related to toolbox path caching (open menu File/Preferences/General and run "Update Toolbox Path Cache")
7. Removed your old YALMIP version from the MATLAB path?
8. Added all the paths to your [solver](/allsolvers)?
9. Do you have any [solver](/allsolvers) installed? (YALMIP does not have any internal low-level solvers
10. Compiled the solver (if needed)? 
11. Compiled it for the correct MATLAB version?
12. Does the solver even work (without using YALMIP)
13. Using MATLAB 7 or later? Some parts work in earlier versions, but not all, and no testing is done on old versions.
14. Probably a PICNIC problem ;-P. Contact me for support.
