---
title: "Installation"
layout: single
sidebar:
  nav: "tutorials"
---

YALMIP is entirely based on m-code, and is thus easy to install. Remove any old version of YALMIP, unzip the downloaded zip-file  and **add the following directories to your MATLAB path**

````
->/yalmip
->/yalmip/extras
->/yalmip/solvers
->/yalmip/modules
->/yalmip/modules/parametric
->/yalmip/modules/moment
->/yalmip/modules/global
->/yalmip/modules/sos
->/yalmip/operators
````

A lazy way to do this is `addpath(genpath(yalmiprootdirectory))`

To test your installation, run the command [yalmiptest]. For further examples and tests, run code from this manual!

### Solvers

YALMIP is not shipped with any low-level solvers. [Solvers](/yalmip/solvers) should be installed as described in the solver manuals. Make sure to add required paths. Your MATLAB installation might already have solvers available that YALMIP will interface, but make sure you understand which solvers you are using, and read about their expected performance [here](/yalmip/solvers).

### Common issues

If you have problems, please read the [FAQ](yalmip/faq).

If you have [MPT] installed, make sure that you delete the YALMIP distribution residing inside [MPT] and remove the old path definitions. Better, don't install YALMIP manually but use [MPTs toolbox manager](http://tbxmanager.com)

If you have used YALMIP before, type `clear classes` or restart MATLAB before using the new version.

YALMIP is primarily developed on a Windows machine using MATLAB 2015a. The code should work on any platform, but is developed and thus most extensively tested on Windows. Most parts of YALMIP should in principle work with MATLAB 6.5, but has not been tested (to any larger extent) on these versions. MATLAB 5.2 or earlier versions are definitely not supported.
