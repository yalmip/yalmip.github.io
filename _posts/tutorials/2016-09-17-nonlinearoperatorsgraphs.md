---
title: "Nonlinear operators - graphs and conic models"
category: tutorial
author_profile: false
level: 4
tags: [Epigraph, Hypograph, Conic programming representable, Linear programming representable, Second-order cone programming representable, Semidefinite programming representable, Exponential cone programming representable]
excerpt: "Epi- and hypograph conic representations of nonlinear operators"
layout: single
header:
  teaser:
sidebar:
  nav: "tutorials"
---

YALMIP supports modeling of nonlinear, often non-differentiable, operators that typically occur in convex programming. Some examples are [min](/command/min), [max](/command/max), [abs](/command/abs), [geomean](/command/geomean), [sumabsk](/command/sumabsk), and [sqrt](/command/sqrt), and users can easily add their own (see the end of this page). The operators can be used intuitively, and YALMIP will automatically try to find out if they are used in a way that enables a conic representation or graph representation. If such representation is impossible, YALMIP automatically tries to revert to a [mixed-integer representation](/tutorial/nonlinearoperatorsmixedinteger).

Although the automatic support for these operators can simplify the modeling phase significantly in some cases, it is recommended to use these operators only when you know how to model them your self using epigraphs and composition rules of convex and concave functions, why and when it can be done etc. The text-book [Boyd and Vandenberghe 2004](/reference/boyd2004) should be a suitable introduction for the beginner, and is consistent with the notation used here.

### Convexity analysis

Without going into theoretical details, the convexity analysis is based on epi- and hypograph formulations, and composition rules. For the composite expression \\(f = h(g(x))\\), it holds that (For simplicity, we write increasing, decreasing, convex and concave, but the correct notation would be nondecreasing, nonincreasing, convex or affine and concave or affine. This notation is used throughout this manual and inside YALMIP)

1. \\(f\\) is convex if \\(h\\) is convex and increasing and \\(g\\) is convex
2. \\(f\\) is convex if \\(h\\) is convex and decreasing and  \\(g\\) is concave
3. \\(f\\) is concave if \\(h\\) is concave and increasing and \\(g\\) is concave
4. \\(f\\) is concave if \\(h\\) is concave and decreasing and \\(g\\) is convex

Based on this information, it is possible to recursively analyze convexity of a complex expression involving convex and concave functions. When [optimize](/command/optimize) is called, YALMIP checks the convexity of objective function and constraints by using information about the properties of the operators. If YALMIP manage to prove convexity, graph formulations of the operators are automatically introduced. This means that the operator is replaced with a new variable, and a set of constraints are added to the model.

**epigraph:**  \\(t\\) replaces convex function \\(f(x)\\) : introduce with \\(f(x) \leq t\\)

**hypograph:** \\(t\\) replaces concave function \\(f(x)\\) : introduce with \\(f(x) \geq t\\)

Of course, in order for this to be useful, the epigraph representation has to be possible to simplify, preferably with a conic constraint, otherwise nothing is gained. As an example, given the model \\( \left\lvert x \right\rvert \leq 1\\), this passes convexity analysis, and a (redundantly large) equivalent model is  \\(\left\lvert x\right\rvert \leq t, t\leq 1\\), which can be represented using the linear constraints \\(-t \leq  x \leq t, t\leq 1\\). Of course, the variable \\(t\\) can be eliminated from the model, but for more complex models it is preferable to keep the intermediate graph variables to simplify manipulations. 

### Standard use

Consider once again the linear regression problem.

````matlab
a = [1 2 3 4 5 6]';
t = (0:0.2:2*pi)';
x = [sin(t) sin(2*t) sin(3*t) sin(4*t) sin(5*t) sin(6*t)];
y = x*a+(-4+8*rand(length(x),1));
a_hat = sdpvar(6,1);
residuals = y-x*a_hat;
````

Using [abs](/command/abs) and [max](/command/max), we can easily solve the \\( L_1 \\) and the \\( L_{\infty} \\) regression problem (Note it is much more efficient to use the [norm](/command/norm) operator than using a sum of [abs](/command/abs) as YALMIP has to reason individually around every element in the absolute value whereas the [norm](/command/norm) operator only has one output). Explicitly creating absolute values when minimizing the \\(L_1\\)  error is simply unnecessarily complicated, but we use it here to illustrate the use of composite operations.

````matlab
optimize([],sum(abs(residuals)));
a_L1 = value(a_hat)
optimize([],max(abs(residuals)));
a_Linf = value(a_hat)
````

YALMIP automatically concludes that the objective functions can be modeled using graph models based on linear inequalities, adds these, and solves the problems. We can simplify the code even more by using the [norm](/command/norm) operator. Here we also compute the least-squares solution (note that this norm, as written here, will generate a second-order cone constraint, arising when the graph model for the second-order cone representable 2-norm is modeled).

````matlab
optimize([],norm(residuals,1));
a_L1 = value(a_hat)
optimize([],norm(residuals,2));
a_L2 = value(a_hat)
optimize([],norm(residuals,inf));
a_Linf = value(a_hat)
````

The following piece of code shows how we easily can solve a regularized problem.

````matlab
optimize([],1e-3*norm(a_hat,2)+norm(residuals,inf));
a_regLinf = value(a_hat)
````

The [norm](/command/norm) operator is used exactly as the built-in [norm](/command/norm) function in MATLAB, both for vectors and matrices. Not only vector norms are conic representable, but also the largest singular value (2-norm in matrix case) and the Frobenious norm of a matrix are, to name a few.

A construction useful for maximizing determinants of positive definite matrices is the function \\( \det (P)^{1/m}\\), for positive definite matrix \\( P \\), where \\( m \\) is the dimension of \\(P\\). This concave function, called [geomean](/command/geomean) in YALMIP, is supported as an operator. Note that the positive semidefiniteness constraint on \\( P \\) is added automatically by YALMIP.

````matlab
D = randn(5,5);
P = sdpvar(5,5);
optimize([P <= D*D'],-geomean(P));
````

The command can be applied also on positive vectors, and will then model the geometric mean of the elements. We can use this to find the analytic center of a set of linear inequalities (note that this is absolutely not the recommended way to compute the analytic center.)

````matlab
A = randn(15,2);
b = rand(15,1)*5;
x = sdpvar(2,1);
optimize([],-geomean(b-A*x)); % Maximize product of elements in b-Ax, s.t Ax < b
````

Rather advanced constructions are possible, and YALMIP will try derive an equivalent convex model.

````matlab
sdpvar x y z
F = [max(1,x)+max(y^2,z) <= 3, max(1,-min(x,y)) <= 5, norm([x;y],2) <= z];
sol = optimize(F,max(x,z)-min(y,z)-z);
````

### Polynomial and sigmonial expressions

By default, polynomial expressions (except quadratics) are not analyzed with respect to convexity and conversion to a conic model is not performed. Hence, if you add a constraint such as \\(x^4 + y^8-x^{0.5} \leq 10\\), YALMIP may complain about convexity, even though we can see that the expression is convex and can be represented using conic constraints. More importantly, YALMIP will not try to derive an equivalent conic model. However, by using the command [cpower](/command/cpower) instead, (rational) powers can be used.

To illustrate this, first note the difference between a monomial generated using overloaded power and a variable generated using [cpower](/command/cpower).

````matlab
sdpvar x
x^4
Polynomial scalar (real, homogeneous, 1 variable)
cpower(x,4)
Nonlinear scalar (real, 1 variable)
````

Working with these convexity-aware monomials is no different than usual.

````matlab
sdpvar x y
F = [cpower(x,4) + cpower(y,4) <= 10, cpower(x,2/3) + cpower(y,2/3) >= 1];
plot(F,[x y]);
````

Note that when you plot sets with constraints involving nonlinear operators and polynomials, it is recommended that you specify the variables of interest in the second argument (YALMIP may otherwise plot the set with respect to auxiliary variables introduced during the construction of the conic model.)

Do not use these operators unless you really need them. The conic representation of rational powers easily grow large.

### A look behind the scene

If you want to look at the model that YALMIP generates, you can use the two commands **model** and **expandmodel**. Please note that these expanded models never should be used manually. The commands described below should only be used for illustrating the process that goes on behind the scenes.

With the command model, the epi- or hypograph model of the variable is returned together with a struct that describes the operator. As an example, to model the maximum of two scalars x and y, YALMIP generates two linear inequalities.

````matlab
sdpvar x y
t = max([x y]);
[operator,F] = model(t)
++++++++++++++++++++++++++++++++++++++++++++
|   ID|      Constraint|               Type|
++++++++++++++++++++++++++++++++++++++++++++
|   #1|   Numeric value|   Element-wise 1x2|
++++++++++++++++++++++++++++++++++++++++++++
sdisplay(sdpvar(F(1)))
ans =
   '-x+t'    '-y+t'

operator =

       convexity: 'convex'
    monotonicity: 'increasing'
    definiteness: 'none'
            name: 'max'
          models: 1313
      convexhull: []
          bounds: []
           range: [-Inf Inf]
          domain: [-Inf Inf]
      derivative: []
           model: 'graph'
````

For more advanced models with recursively used nonlinear operators, the function model will not generate the complete model since this low-level function does not expand the arguments. For this case, use the command **expandmodel**. This command takes two arguments, a set of constraints and an objective function. To expand an expression, just let the expression take the position as the objective function. Note that the command assumes that the expansion is performed in order to prove a convex function, hence if you expression is meant to be concave, you need to negate it. To illustrate this, let us expand the objective function in an extension of the geometric mean example above.

````matlab
A = randn(7,2);
b = rand(7,1)*5;
x = sdpvar(2,1);
expandmodel([],-geomean([b-A*x;min(x)]))
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
|   ID|      Constraint|                               Type|                    Tag|
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
|   #1|   Numeric value|         Element-wise (derived) 2x1|       Expansion of min|
|   #2|   Numeric value|   Second order cone constraint 3x1|   Expansion of geomean|
|   #3|   Numeric value|   Second order cone constraint 3x1|   Expansion of geomean|
|   #4|   Numeric value|   Second order cone constraint 3x1|   Expansion of geomean|
|   #5|   Numeric value|   Second order cone constraint 3x1|   Expansion of geomean|
|   #6|   Numeric value|   Second order cone constraint 3x1|   Expansion of geomean|
|   #7|   Numeric value|   Second order cone constraint 3x1|   Expansion of geomean|
|   #8|   Numeric value|   Second order cone constraint 3x1|   Expansion of geomean|
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
````

The result is two linear inequalities related to the min operator and 7 second order cone constraints used for the conic representation of the geometric mean.

### Adding new operators

If you want to add your own operator, all you need to do is to create 1 file. This file should be able to return the numerical value of the operator for a numerical input, and return the epigraph (or hypograph) and a descriptive structure of the operator when the first input is **'graph'**. As an example, the following file implements the nonlinear operator tracenorm. This convex operator returns **sum(svd(X))** for matrices **X**. This value can also be described as the minimizing argument of the optimization problem

$$
\textbf{min}_{t,A,B}  ~ \textbf{  subject to } \begin{bmatrix}A & X\\X^T & B\end{bmatrix}, \textbf{tr}(A)+\textbf{tr}(B) \leq 2t
$$

The code looks essentially the same for all conic representable operators.

````matlab
function varargout = tracenorm(varargin)

switch class(varargin{1})    

    case 'double' % What is the numerical value (needed for displays etc)
        varargout{1} = sum(svd(varargin{1}));

    case 'char'   % YALMIP send 'graph' when it wants the epigraph or hypograph
        switch varargin{1}
	  case 'graph'
            t = varargin{2}; % 2nd arg is always the extended operator variable
            X = varargin{3}; % 3rd arg and above are always arguments user used.
            A = sdpvar(size(X,1));
            B = sdpvar(size(X,2));
            F = [[A X;X' B] >= 0, trace(A)+trace(B) <= 2*t];

            % Operator description
            properties.convexity    = 'convex';   % convex | none | concave
            properties.monotonicity = 'none';     % increasing | none | decreasing
            properties.definiteness = 'positive'; % negative | none | positive  
            properties.model        = 'graph';    % Redundnant here...

            % Return epigraph model, properties, and input arguments
            varargout{1} = F;            
            varargout{2} = properties;
            varargout{3} = X;

         case 'exact'
            % Exact representation based on integer programming not available
            % We could add a callback logic, but we keep it simple
	          varargout{1} = [];
	          varargout{2} = [];
	          varargout{3} = [];

         otherwise
            error('Something is very wrong now...')
        end    

    case 'sdpvar' % Always the same for R^n -> R^1
        varargout{1} = yalmip('define',mfilename,varargin{:});    

    otherwise
end
````

Additional properties of the operator can be assigned to guide YALMIP in various scenarios. However, most those additional properties are not of interest for graph-based implementations, but are meant for [mixed-integer](/tutorial/nonlinearoperatorsgraphs) and [callback-based](tutorial/nonlinearoperatorscallback) representations.

The function **sumk** in YALMIP is implemented using this framework and might serve as a good start for your own experiments.
