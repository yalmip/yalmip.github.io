---
title: "Nonlinear operators - integer models"
permalink: "/tutorial/nonlinearoperatorsmixedinteger"
category: inside
subcategory: 1
tags: [Integer programming representable]
excerpt: "Mixed-integer representations of nonlinear operators"
---

In addition to modeling convex and concave operators and perform automatic analysis and derivation of equivalent conic programs using [graph models](/tutorial/nonlinearoperatorsgraphs), YALMIP uses the nonlinear operator framework for implementing logic and combinatorial expression involving commands such as [or](/command/or), [and](/command/and), [ne](/command/ne), [iff](/command/iff), [implies](/command/implies), [nnz](/command/nnz), [alldifferent](/command/alldifferent), [sort](/command/sort) and [ismember](/command/ismember), and on a higher level, nonconvex piecewise functions in connection with [MPT](/solver/mpt). The common feature among these operators is that they all require binary and integer variables to be represented in a structured way.

The same framework is used also for alternatives to graph-based implementations. If the convexity propagation of a conic representable function such as [min](/command/min) or [max](/command/max) fails, thus invalidating the use of graph-models, YALMIP can create an alternative model based on mixed-integer representations. This done for many of the [linear programming representable operators](/tags/#linear-programming-representable).

Mixed-integer representations are also used to model discontinuous functions such as [floor](/command/floor), [ceil](/command/ceil), [fix](/command/fix), [round](/command/round), [sign](/command/sign), [rem](/command/rem), and [mod](/command/mod).

### Working with mixed-integer representations

Consider the following simple example which violates propagation rules for convexity. YALMIP will detect this, and switch to a mixed-integer representation of the absolute value. The end result is a mixed-integer linear program.

````matlab
sdpvar x y
F = [abs(abs(x+1)+3) >= y, 0<=x<=3];
sol = optimize(F,-y);
value([x y])
ans =
    3.0000    7.0000
````

Since the mixed-integer models are based on big-M reformulations, it is crucial that you have explicit bounds on all variables involved in the nonconvex expressions. Read more about this in the [big-M tutorial](/tutorial/bigmandconvexhulls).

If you not want YALMIP to resort to mixed-integer models in nonconvex cases, you can turn off this feature

````matlab
sdpvar x y
F = [abs(abs(x+1)+3) => y, 0<=x<=3];
sol = optimize(F,-y,sdpsettings('allownonconvex',0));
sol.info

ans =

Convexity check failed (Expected concave function in constraint #1 at level 1)
````


### Mixed-integer model as alternatives

We will start by implementing a rudimentary representation of scalar absolute value, with support for both a graph-model and an integer model. The difference compared to the model we created in [graph-representation] is that we return a mixed-integer model when YALMIP asks for an exact model. The hard part is of course to come up with a suitable [integer model](/tutorial/bigmandconvexhulls). Notice the use of the function **derivebounds**, which will give us bounds on the argument, thus helping us to obtain a numerically sound  [big-M](/tutorial/bigmandconvexhulls) model (assuming that explicit bounds have been added to the involved variables in the model)

````matlab
function varargout = abs(varargin)
switch class(varargin{1})    

    case 'double'
        error('This should have been caught by built-in.')

    case 'char'   
        switch varargin{1}
          case 'graph'
            t = varargin{2};
            X = varargin{3};

            F = [-t <= x <= t];

            properties.convexity    = 'convex';
            properties.monotonicity = 'none';  
            properties.definiteness = 'positive';
            properties.model        = 'graph';	  

            varargout{1} = F;
            varargout{2} = properties;
            varargout{3} = X;

          case 'exact'

            t = varargin{2};
            X = varargin{3};

            [M,m] = derivebounds(X);
            z = binvar(1);
            F = [0 >= x - t >= 2*m*z,
                        m*z <= x,
                 0 <= t + x <= 2*M*(1-z),
                          x <= M*(1-z)];

            properties.convexity    = 'convex';
            properties.monotonicity = 'none';
            properties.definiteness = 'positive';	  
            properties.model        = 'integer';

            varargout{1} = F;
            varargout{2} = properties;
            varargout{3} = X;

          otherwise
            error('Something is very wrong now...')
        end    

    case 'sdpvar' % Always the same for R^n -> R^1
        varargout{1} = yalmip('define',mfilename,varargin{:});    

    otherwise
end

````

### Mixed-integer models as default

Some operators, such as [sign](/command/sign), does not have a graph-representation, but must be modelled using an integer representation (or a [callback approach](/tutorial/nonlinearoperatorscallback)).

In these cases, we create an operator that always returns the mixed-integer model, even though YALMIP asks for a graph-model. We communicate the fact that we returned a mixed-integer model via the model field in the properties. By returning an integer model directly instead of simply returning an empty model when YALMIP asks for a graph-model, we reduce the work-load for YALMIP (if YALMIP fails to get a graph model, it will make a second call and ask for an exact model, unless an exact model was returned anyway).

````matlab
function varargout = sign(varargin)
switch class(varargin{1})    

    case 'double'
        error('This should have been caught by built-in.')

    case 'char'   

        t = varargin{2};
        X = varargin{3};

        d = binvar(1,1);
        [M,m] = derivebounds(X);
        F = [X >= d*m,X <=(1-d)*M, t == 1-2*d];

        properties.convexity    = 'none';
        properties.monotonicity = 'increasing';
        properties.definiteness = 'none';	  
        properties.model        = 'integer';	  

        varargout{1} = F;
        varargout{2} = properties;
        varargout{3} = X;

    case 'sdpvar' % Always the same for R^n -> R^1
        varargout{1} = yalmip('define',mfilename,varargin{:});    

    otherwise
end
````
