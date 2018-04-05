---
title: "Nonlinear operators - callbacks"
category: tutorial
author_profile: false
level: 4
tags:
excerpt: "Callback representations of nonlinear operators"
layout: single
header:
  teaser:
sidebar:
  nav: "tutorials"
---

The nonlinear operator framework was initially implemented for functions that can be modelled rigorously using conic constraints and additional variables through [graph representations](/tutorial/nonlinearoperatorsgraphs).

However, there are many functions that cannot be modelled using conic constraints, such as exponential functions and logarithms, but are convex or concave, and of course can be analyzed in terms of convexity preserving operations. These function (and any other nonlinear function) are supported in a framework called callback nonlinear operators. Models using these general operators will still be analyzed with respect to convexity, but the resulting model requires a general nonlinear solver, such as [FMINCON](/solver/fmincon) or [SNOPT](/solver/snopt).

In addition to convexity properties and simple function values, operators can be attributed with various other properties, and we will use this possibility here. Examples include derivative information and envelope approximators for the global solver [BMIBNB](/solver/bmibnb).

Note that the framework currently is focused on elementwise functions or functions with vector input and scalar output. The tutorial here concentrates on the elementwise case. 

### Standard use

Essentially all simple univariate functions in MATLAB are available as callback operators in YALMIP.

The following problem should return 2, if the nonlinear solver you use (probably [FMINCON](/solver/fmincon)) performs well.

````matlab
sdpvar x
optimize([],-exp(-(x-2).^2));
value(x)
````

Note that the problem is nonconvex, but is nevertheless solved. If we want to exit if YALMIP detects nonconvexity (or more precisely fails to prove convexity), we must tell YALMIP this.

````matlab
sdpvar x
sol = optimize([],-exp(-(x-2).^2),sdpsettings('allownonconvex',0));
sol.info

ans =
 'Convexity check failed (Expected convexity in objective at level 1)'
````

Since the problem is nonconvex, we cannot be sure that the computed solution actually is a global minimizer. An alternative then is to invoke the built-in global solver [BMIBNB](/solver/bmibnb) (or an external global solver such as [BARON](/solver/baron) or [SCIP](/solver/scip)). Global solutions are typically extremely time-consuming, but this trivial problem is solved immediately.

````matlab
optimize([-5 <= x <= 5],-exp(-(x-2).^2),sdpsettings('solver','bmibnb'));
````

The global solver [BMIBNB](/solver/bmibnb) is based on the envelope outer approximations discussed below. Note that the solver requires bounds (preferably explicit) on variables that are involved in nonconvex terms. The reason for this can be found in the [tutorial on envelopes and global optimization](/tutorial/envelopesinbmibnb) and the [big-M tutorial](/tutorial/bigmandconvexhulls).

In general, working with nonlinear callback-based operators in YALMIP requires no special code. It must however be kept in mind that they require general-purpose nonlinear solvers.

Another issue to keep in mind is that the operators typically normalize the problem in some sense. Operators are only applied to simple variables. In other words, auxiliary variables are introduced to put the model in a canonical form. Hence, the problem that is solved above, will internally be converted to the following model.

````matlab
sdpvar x y
optimize([-5 <= x <= 5,y == -(x-2).^2],-exp(y),sdpsettings('solver','bmibnb'));
````

This can cause problems for some solvers if they perform badly on nonconvex equalities.


### Adding new operators

Almost all built-in operators are already supported, but it might happen that you need to implement your own functionality (if you are lazy, you can often use the [sdpfun](/command/sdpfun) functionality).

To illustrate how this is done, we will work with the implementation of **exp**. Since almost all built-in functions in MATLAB behaves the same, the definition can almost always be done in the same way, and to simplify coding , the command **InstantiateElementWise** is available (which hides code very similiar to the code used in the [graph representations]. This sets up the logic in YALMIP for the definition of an operator that works element-wise. As in the other modelling approaches, three outputs are assumed; a set of constraints, typically domain constraints, an operator description, and the input arguments.

````matlab
function varargout = exp(varargin)
switch class(varargin{1})

    case 'double'
        error('Should be caught by built-in.')

    case 'sdpvar'
        varargout{1} = InstantiateElementWise(mfilename,varargin{:});

    case 'char'

        X = varargin{3};

        F = [];

        operator.convexity    = 'convex';
        operator.monotonicity = 'increasing';
        operator.definiteness = 'positive';
        operator.model =        'callback';

        varargout{1} = F;
        varargout{2} = operator;
        varargout{3} = X;

    otherwise
        error('SDPVAR/EXP called with CHAR argument?');
end

````

At this point we have a model that can be used in, e.g., [FMINCON](/solver/fmincon). The convexity information lets YALMIP perform convexity analysis of expressions involving the exponential function.

The function can also be used when using the global solver [BMIBNB](/solver/bmibnb), as we did above. However, the global solver is based on function bounding, and approximating nonlinear expressions locally by convex hulls. To enable this, YALMIP must have more information. If no information is available, YALMIP uses an approximate sampled-based scheme, which is both slow and theoretically questionable.

Hence, we need to add two more properties to the operator. One function, the bounding operator, which returns lower and upper bounds on the nonlinear function, given lower and upper bounds on the argument, and a function that returns the envelope (or outer approximation of it) in a particular form.

The improved version is now

````matlab
function varargout = exp(varargin)
switch class(varargin{1})

    case 'double'
        error('Should be caught by built-in.')

    case 'sdpvar'
        varargout{1} = InstantiateElementWise(mfilename,varargin{:});

    case 'char'

        X = varargin{3};

        F = [];

        operator.convexity    = 'convex';
        operator.monotonicity = 'increasing';
        operator.definiteness = 'positive';
        operator.convexhull   = @convexhull;
        operator.bounds       = @bounds;
        operator.derivative   = @derivative;
        operator.model =      'callback';

        varargout{1} = F;
        varargout{2} = operator;
        varargout{3} = X;

    otherwise
        error('SDPVAR/EXP called with CHAR argument?');
end

function [L,U] = bounds(xL,xU)
L = exp(xL);
U = exp(xU);

function df = derivative(x)
df = exp(x);

function [Ax, Ay, b] = convexhull(xL,xU)
fL = exp(xL);
fU = exp(xU);
dfL = exp(xL);
dfU = exp(xU);
[Ax,Ay,b] = convexhullConvex(xL,xU,fL,fU,dfL,dfU);

````

The global solver [BMIBNB](/solver/bmibnb) can now quickly obtain linear envelope approximations of the operator. The derivative information can be used by YALMIP to perform automatic differentiation when the nonlinear solver requests derivatives of objective functions and constraints.

Note that some of the information is redundant. Since we have described that the operator is increasing, it immediately follows that YALMIP can derive lower and upper bounds over an interval. Likewise, since we have defined the function as convex, and supply a callback for computing the derivative, YALMIP can automatically generate the convex hull approximation. It is however recommended to explicitly state as much as possible manually, since this will speed up the code, and avoid problem with, e.g., functions that are undefined or infinite  on the border of its domain.
