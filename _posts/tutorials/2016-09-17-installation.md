---
title: "Installation"
category: tutorial
author_profile: false
tags: 
excerpt: "If it's hard, you're doing it wrong."
layout: single
level: 0
sidebar:
  nav: "tutorials"
---

YALMIP is entirely based on m-code, and is thus easy to install.

The official version can be found at https://github.com/yalmip/yalmip/archive/master.zip. In some cases you might need the most recent development branch, and this can be found at https://github.com/yalmip/yalmip/archive/develop.zip (don't install this unless you absolutely have to for some reason, and you actually know what you are doing).

Remove any old version of YALMIP, unzip the downloaded zip-file  and **add the following directories to your MATLAB path**

````
->/YALMIP-master
->/YALMIP-master/extras
->/YALMIP-master/solvers
->/YALMIP-master/modules
->/YALMIP-master/modules/parametric
->/YALMIP-master/modules/moment
->/YALMIP-master/modules/global
->/YALMIP-master/modules/sos
->/YALMIP-master/operators
````

Of course, you do not have to call the directory YALMIP-master, that just happens to be the name of the zip that Github generates for the master branch..

A lazy way to do this is `addpath(genpath(yalmiprootdirectory))`

If you want to be even lazier, simply run the following code in the directory where you want to install YALMIP.

````matlab
cd YALMIPfolderShouldbeHere
urlwrite('https://github.com/yalmip/yalmip/archive/master.zip','yalmip.zip');
unzip('yalmip.zip','yalmip')
addpath(genpath([pwd filesep 'yalmip']));
savepath
````

Another approach is to handle your installation using [www.tbxmanager.com](http://tbxmanager.com)

To test your installation, run the command [yalmiptest](/command/yalmiptest). For further examples and tests, run code from this manual!

If things fail or you suspect there is some problem, solve a trivial problem and see what happens.

````matlab
% Does YALMIP work at all? If not, we might not even be able to create a variable
x = sdpvar(x)

% Can any solver be called?
optimize(x>= 0, x,sdpsettings('debug',1))

% Can the solver you have troubles with be called?
optimize(x>= 0, x,sdpsettings('debug',1,'solver','thissolver'))
````

### Solvers

YALMIP is not shipped with any low-level solvers. [Solvers](/allsolvers) should be installed as described in the solver manuals. Make sure to add required paths. Your MATLAB installation might already have solvers available that YALMIP will interface, but make sure you understand which solvers you are using, and read about their expected performance [here](/allsolvers).

### Common issues

If you have problems, please read the [FAQ](/faq).

If you have [MPT](/solver/mpt) installed, make sure that you delete the YALMIP distribution residing inside [MPT](/solver/mpt) and remove the old path definitions. Better, don't install YALMIP manually but use [MPTs toolbox manager](http://tbxmanager.com)

If you have used YALMIP before, type `clear classes` or restart MATLAB before using the new version.

YALMIP is primarily developed on a Windows machine using MATLAB 2015a. The code should work on any platform, but is developed and thus most extensively tested on Windows. Most parts of YALMIP should in principle work with MATLAB 6.5, but has not been tested (to any larger extent) on these versions. MATLAB 5.2 or earlier versions are definitely not supported.
