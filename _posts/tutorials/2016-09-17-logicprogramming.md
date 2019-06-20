---
title: "Logics and integer-programming representations"
category: tutorial
author_profile: false
level: 4
tags: [Logic programming]
excerpt: "Logic programming in YALMIP means programming with operators such as alldifferent, number of non-zeros, implications and similiar combinatorial objects."
layout: single
sidebar:
  nav: "tutorials"
---


Some simple rules and strategies when deriving models for complex logic and combinatorials models:
1. Try to represent things with disjoint events of the kind "exactly one of these things occur" with a binary variable associated to each event.
2. Try to arrive at "binary variable implies set of constraints"
3. Introduce intermediate auxilliary variables to keep things clean
4. Decompose the logic using intermediate variables to connect parts

In other words, a large sparse simple model is much better than a compact complex model.

Throughout the examples here, **a** and **b** represent scalar binary variables, **z** represents a vector of binary variables, and **x** is just some generic variable (could be anything).

Note that what we try to emphasize in the examples is that most models can be derived using exactly the same strategy and basic building blocks. Many of the models can be simplified and reduced, but those steps are typically done just as efficiently inside the solver during pre-solve.

In the models, we make use of [big-M models](/tutorial/bigmandconvexhulls/) with associated constant \\(M\\). We use a notation with the same value everywhere, but in practice you would spend effort on deriving as small constants as possible for every single constraint.

## Logical models involving binary variables

### s = NOT a

With binary \\(a = 1\\) representing true and \\(a = 0\\) representing false, logical negation turns into 

$$
s = 1-a
$$


### s = a AND b

\\(s\\) has to be \\(1\\) if both \\(a\\) and \\(b\\) are 1. \\(s\\) has to be \\(0\\) if  either of \\(a\\) and \\(b\\) are 0.

$$
s \geq a + b -1,~s \leq a,~s\leq b
$$

The idea generalizes to an arbitrary number of binary variables \\(s = z_1 ~\&~ z_2~ \&~ \ldots ~\&~ z_n\\) with

$$
s \geq \sum_{i=1}^n z_i - (n-1), ~s\leq z
$$

### s = a OR b

\\(s\\) has to be \\(1\\) if  either of \\(a\\) and \\(b\\) are 1, and \\(0\\) if none of them are 1.

$$
s \geq a,~s\geq b, ~ s \leq a + b 
$$

### s = a XOR b

\\(s\\) has to be \\(1\\) if  exactly one of \\(a\\) and \\(b\\) are 1, and \\(0\\) otherwise

$$
s \geq a - b, ~s \geq b-a, ~ s \leq a + b, ~ a + b\leq 2-s
$$

The idea generalizes to an arbitrary number of binary variables  with

$$
s \geq z_i - \sum_{j\ne i} z_j,~ s\leq \sum_{i = 1}^n z_j \leq n + (1-n)s
$$


### If a then b

Logical implication of binary variables


$$
b \geq a
$$


## Logical models involving binary variables and constraints

### If a then  \\(f(x)\leq 0\\)

To ensure a constraint holds when a binary is true, we model implication using a [big-M strategy](/tutorial/bigmandconvexhulls/).

$$
f(x) \leq M(1-a)
$$

This is the core model construct which is used for almost all cases below.

### If logical(z) then  \\(f(x)= 0\\)

Introduce a new binary variable \\(s\\) to represent the logical condition using the methods above, and then use the standard implication.

This is a general strategy throughout. If you enter complex expressions, introduce new variables for simple sub-parts and build the complete model by combining simple standard models.

### If a then  \\(f(x) < 0\\)

Strict inequalities are impossible in practice (unless \\(f(x)\\) is quantized such as only involving integer variables). Hence, you have to use a margin and hope that this in combination with solver tolerances leads to a solution which actually satisfies the constraints (solutions claimed feasible and optimal can easily be infeasible and are only guaranteed to satisfy solver tolerance and termination critera)

$$
f(x) \leq -\epsilon  + M(1-a)
$$

### If a then  \\(f(x)\leq 0\\) else  \\(f(x)\geq 0\\)

The standard implication is simply repeated for the two cases.

$$
f(x) \leq M(1-a),-f(x)\leq Ma
$$

To create more easily generalizable models and learn a common core strategy, it is adviced to think of this as two disjoint cases each associated with a set of constraints. 

$$
\begin{align}
z_1 &\rightarrow \{f(x)\leq 0, ~ a=1\}\\
z_2 &\rightarrow \{-f(x)\leq 0, ~ a=0\}\\
z_1 + z_2 &= 1
\end{align}
$$

Once again ending up as standard models for implications

$$
\begin{align}
f(x)&\leq M(1-z_1), -(1-z_1)\leq a-1 \leq (1-z_1)\\
-f(x) &\leq M(1-z_2), -(1-z_2) \leq a \leq (1-z_2)\\
z_1 + z_2 &= 1
\end{align}
$$

### If a then  \\(f(x)\leq 0\\) else  \\(g(x)\leq 0\\)

We could of course create  a model using \\(a\\) only, but using the same strategy the whole time trains us to realize that all model look the same in principle

$$
\begin{align}
z_1 &\rightarrow \{f(x)\leq 0, ~ a=1\}\\
z_2 &\rightarrow \{g(x)\leq 0, ~ a=0\}\\
z_1 + z_2 &= 1
\end{align}
$$

Standard models for implications

$$
\begin{align}
f(x) &\leq M(1-z_1), -(1-z_1)\leq a-1 \leq (1-z_1)\\
g(x) &\leq M(1-z_2), -(1-z_2) \leq a  \leq (1-z_2)\\
z_1 + z_2 &= 1
\end{align}
$$

### If a then  \\(f(x)= 0\\)

This is nothing but  two inequalities in the implication, hence

$$
\begin{align}
f(x)&\leq M(1-a)\\
-f(x) &\leq M(1-a)\\
\end{align}
$$

### If \\( f(x) \leq 0\\) then a

When \\(f(x)\\) becomes negative, the binary variable should be forced to be activated. This is accomplished by reversing the constraint and introducing an implication for the reverse model

$$
f(x)\geq -M(1-a)
$$

Note that we use a non-strict inequality. If behaviour around \\f(x)\\) is important, a margin will have to be used as discussed before.


### If \\( f(x) = 0\\) then a

First, this is extremely ill-posed in practice as solvers work with floating-point numbers so it might consider, e.g., \\(10^{-7\\) to be 0. It is also often ill-posed from a practical point of view. If your model says **if waterlevel is 0** do you really want it to behave differently compared to the case when the waterlevel is \\(10^{-11}\\), i.e. the thickness of one atom?

The disjoint logical model is
$$
f(x)<0 \rightarrow a = 0
f(x)=0 \rightarrow a = 1
f(x)>0 \rightarrow a = 0
$$

This is interpreted as 

$$
\begin{align}
z_1 &\rightarrow \{f(x)<0, a=0\} \\
z_2 &\rightarrow \{f(x)=0, a=1\} \\
z_3 &\rightarrow \{f(x)>0,a=0\} \\
z_1+z_2+z_3 &= 1
\end{align}
$$

A big-M representation of the implications, using a margin \\(\epsilon\\) around 0 if wanted leads to

$$
\begin{align}
-(1-z_1) &\leq a \leq (1-z_1), f(x) \leq -\epsilon + M(1-z_1)\\
-(1-z_2) &\leq a-1 \leq (1-z_2), -M(1-z_2)-\epsilon \leq f(x) \leq \epsilon + M(1-z_2)\\
-(1-z_3) &\leq a \leq (1-z_3), f(x) \geq \epsilon -M(1-z_3)\\
z_1+z_2+z_3 &= 1
\end{align}
$$



### If \\( f(x) \leq 0\\) then a else not a

Now it starts paying of thinking in terms of auxilliary variables and disjoint cases. This is (with standard problems arising at the cross-over)

$$
\begin{align}
z_1 &\rightarrow \{f(x)\leq 0, ~ a=1\}\\
z_2 &\rightarrow \{f(x)\geq 0, ~ a=0\}\\
z_1 + z_2 &= 1
\end{align}
$$

A big-M model is thus

$$
\begin{align}
f(x) &\leq M(1-z_1), -(1-z_1)\leq a-1 \leq (1-z_1)\\
g(x) &\leq M(1-z_2), -(1-z_2) \leq a  \leq (1-z_2)\\
z_1 + z_2 &= 1
\end{align}
$$


### If \\( f(x) \leq 0\\) then  \\( g(x) \leq 0\\)

Glue the two conditions using an intermediate binary variable

$$
\begin{align}
f(x)&\leq 0 &\rightarrow z_1\\
z_1 &\implies g(x)\\
\end{align}
$$

These are standard models and we thus have

$$
\begin{align}
f(x)&\geq -M(1-z_1)\\
g(x)&\leq M(1-z_1)\\
\end{align}
$$


## Multiplications of variables and functions

### y = ab, a and b binary

Binary multiplication is nothing but logical and, hence

$$
y \leq a,~ y\leq b,~ y\geq a+b-1
$$

Generalization to more terms follows the same generalization as logical and.

### y = ax, a binary x, x non-binary

Multiplication of binary and non-binary should be seen as a logical operation

$$
\begin{align}
a  &\rightarrow y = x\\
1-a &\rightarrow y = 0\\
\end{align}
$$

This is implemented using our standard model for implications

$$
\begin{align}
-M(1-a) &\leq y-x \leq M(1-a)\\
-Ma &\leq y \leq Ma
\end{align}
$$

### y = abx, a and b binary, x non-binary

When there are repeated binary variables, the procedure is simply repeated with intermediate variables. Start by introducing a new variable and  model for **c = ab**  and then use model for **y = cx**. 

The idea and model generalizes to arbitrary polynomials of binary variables simply introduce intermediate variables to keep it simple.

### y = ax, a binary, x integer

An alternative for multiplying a binary with an integer variable is to make a binary expansion of the integer \\(x = z_1 + 2 z_2+ 4z_3 + \ldots +2^{n+1}z_n\\) with binary variables $z$, and then model new terms \\(y_i = az_i\\) using standard model for binary times binary.

### y = wx, w integer, x continuous 

Make binary expansion of **w** and the create models for binary times continuous.

### y = wx, w integer,  x integer

Make binary expansions of both **w** and **x** and then create models for binary products.

## Representations of functions

### y = abs(x)

A logical representation of absolute value is 

$$
\begin{align}
x &\geq 0 \rightarrow y = x\\
x &\leq 0 \rightarrow y = -x
\end{align}
$$

A disjoint representation of this using binary variables

$$
\begin{align}
z_1 &\rightarrow \{x\geq 0,~y = x\}\\
z_2 &\rightarrow \{x\leq 0,~y = -x\}\\
z_1 + z_2 &= 1
\end{align}
$$

Once again standard implications so the result is a standard big-M model

$$
\begin{align}
-M(1-z_1) &\leq y - x\leq M(1-z_1), x\geq -M(1-z_1)\\
-M(1-z_2) &\leq y + x\leq M(1-z_2), x\leq M(1-z_2)\\
z_1 + z_2 &= 1
\end{align}
$$

Remember that absolute value is convex, so you only use a MILP representation if absoutely needed.

### y = max(x)

The maximum of a vector can also be thought of as a logical model 

$$
\begin{align}
x_1 &\geq x \rightarrow y = x_1\\
\vdots&\\
x_n &\geq x \rightarrow y = x_n
\end{align}
$$

Introduce a binary variable for every case and derive disjoint representation

$$
\begin{align}
z_1 &\rightarrow y = x_1, x_1 \geq x\\
\vdots\\
z_n &\rightarrow y = x_n, x_n \geq x\\
\sum_{i=1}^n z_i &= 1
\end{align}
$$

This is finalized with standard implication models

$$
\begin{align}
-M(1-z_1) &\leq y - x_1\leq M(1-z_1), ~x-x_1 \leq M(1-z_1)\\
\vdots\\
-M(1-z_n) &\leq y - x_1\leq M(1-z_n), ~x-x_n \leq M(1-z_n)\\
\sum_{i=1}^n z_1 &= 1
\end{align}
$$

Remember that max is convex, so you only use a MILP representation if absolutely needed.

### y = min(x)

Use \\( \min(x) = -\max(-x)\\).

### y = sort(x)

Bla bla

### y = floor(x)

In theory introduce integer \\(y\\) and \\(x-1 < y \leq x\\). In practice strict inequality has to be approximated leading to \\(x-1+\epsilon \leq y \leq x\\)

### y = ceil(x)

In theory introduce integer \\(y\\) and \\(x \geq y < x+1\\). In practice strict inequality has to be approximated leading to \\(x \leq y \leq x+1-\epsilon\\)

### y = rem(x,m)

\\( y = rem(x,m)\\) means \\( y = x - n*m, n = \fix(x/m)\\). 

### y = mod(x,m)

Bla bla

### y = sgn(x)

A logical model of \\(s = \sgn(x)\\) is

$$
\begin{align}
x & < 0 \rightarrow s = -1\\
x & = 0 \rightarrow s = 0\\
x & > 0 \rightarrow s = 1\\
\end{align}
$$

This is interpreted as 

$$
\begin{align}
z_1 &\rightarrow \{x<0, s=-1\} \\
z_2 &\rightarrow \{x=0, s=0\} \\
z_3 &\rightarrow \{x>0,s=1\} \\
z_1+z_2+z_3 &= 1
\end{align}
$$

A big-M representation of the implications, using a margin \\(\epsilon\\) around 0 if wanted leads to

$$
\begin{align}
-M(1-z_1) &\leq s+1 \leq M(1-z_1), x \leq -\epsilon + M(1-z_1)\\
-M(1-z_2) &\leq s \leq M(1-z_2), -M(1-z_2)-\epsilon \leq x \leq \epsilon + M(1-z_2)\\
-M(1-z_3) &\leq s-1 \leq M(1-z_3), x \geq \epsilon -M(1-z_3)\\
z_1+z_2+z_3 &= 1
\end{align}
$$


### y = piecewise affine function

Bla bla

### y = piecewise quadratic function

Bla bla
