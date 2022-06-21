---
category: command
excerpt: "Assign numerical values to decision variable"
tags: [Warm-start,Initial,Options, Assign]
date: '2016-09-17'
sidebar:
  nav: "commands"
---

[warmstart](/command/warmstart) is used to explicitly assign the value obtained when applying the command [value](/command/value) on an [sdpvar](/command/sdpvar) object. YALMIP typically assigns values after solving problem, but a manual use of the command is to define initial guesses for [warm-starting](/tags#warm-start) when solvers support these.

Note, warmstart does not replace [replace](/command/replace) or constrain the variable with a particular value. It is only used for initial values in nonlinear solvers. Hence, only used when the option **warmstart** in [sdpsettings](/command/sdpsettings) is activated.

## Syntax

````matlab
warmstart(X,Y)
````

## Examples

Variables are initialized as NaN by default

````matlab
x = sdpvar(1,1);
value(x)

ans =
    NaN
````

By using assign, the current value can be altered

````matlab
warmstart(x,1)
value(x)

ans =
    1
````

By default, inconsistent assignments generate an error message.

````matlab
t = sdpvar(1,1);x = [t t];
warmstart(x,[1 2])
??? Error using ==> sdpvar/warmstart
Inconsistent assignment
````

With a third argument, a least squares assignment is obtained

````matlab
t = sdpvar(1,1);x = [t t];
warmstart(x,[1 2],1)
value(x)

ans =

    1.5000    1.5000
````

The value can be used as an initial guess in warm-starts for solvers supporting this

````matlab
x = sdpvar(1,1);
warmstart(x,pi);
optimize([sin(x)^2 <= .1],x,sdpsettings('warmstart',1));
````
