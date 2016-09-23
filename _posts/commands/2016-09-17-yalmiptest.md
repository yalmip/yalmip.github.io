---
layout: single
category: command
author_profile: false
excerpt: ""
title: yalmiptest
tags:
comments: true
date: '2016-09-17'
sidebar:
  nav: "commands"
---


[yalmiptest](/command/yalmiptest) runs a set of test examples to test the installation.

### Syntax

````matlab
yalmiptest
yalmiptest(ops)
````

### Examples
Run the command without any input to test the default installation.
````matlabb
yalmiptest
````


To test a solver, provide a tag.
````matlabb
yalmiptest('sdpt3')
````

To test a particular setup, provide an option structure.
````matlabb
yalmiptest(sdpsettings('solver','sdpt3'))
````
