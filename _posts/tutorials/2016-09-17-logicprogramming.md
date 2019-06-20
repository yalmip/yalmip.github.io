---
title: "Logic programming"
category: tutorial
author_profile: false
level: 4
tags: [Logic programming]
excerpt: "Logic programming in YALMIP means programming with operators such as alldifferent, number of non-zeros, implications and similiar combinatorial objects."
layout: single
sidebar:
  nav: "tutorials"
---

Throughout the examples here, **a** and **b** represent scalar binary variables, **z** represents a vector of binary variables, and **x** is just some generic variable (could be anything).

Some simple rules and strategies when deriving models for complex logic and combinatorials models:
1. Try to represent things with disjoint events of the kind "exactly one of these things occur" with a binary variable associated to each event.
2. Try to arrive at "binary variable implies set of constraints"
3. Introduce intermediate auxilliary variables to keep things clean
4. Decompose the logic using intermediate variables to connect parts

In other words, a large sparse simple model is much better than a compact complex model.

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


### If a then  \\(f(x)\leq 0\\)

To ensure a constraint holds when a binary is true, we model implication using a big-M strategy

$$
f(x) \leq M(1-a)
$$


### If a then  \\(f(x)\leq 0\\) else  \\(f(x)\geq 0\\)

The standard implication is simply repreated for the two cases.

$$
f(x) \leq M(1-a),-f(x)\leq Ma
$$

To create more easily generalizable models and learn a common core strategy for all models, it is adviced to think of this as two disjoint cases each associated with a set of constraints. 

$$
\begin{align}
z_1 &\rightarrow \{f(x)\leq 0, ~ a=1\}\\
z_2 &\rightarrow \{-f(x)\leq 0, ~ a=0\}\\
z_1 + z_2 &= 1
\end{align}
$$

This can be implemented using standard implications

$$
\begin{align}
f(x)\leq M(1-z_1), -(1-z_1)\leq a-1 \leq (1-z_1)\\
-f(x) \leq M(1-z_2), -(1-z_2) \leq a \leq (1-z_2)\\
z_1 + z_2 &= 1
\end{align}
$$

### If a then  \\(f(x)\leq 0\\) else  \\(g(x)\leq 0\\)

Once again, we make a clean representation using additional variables to encode disjoint cases
$$
\begin{align}
f(x)\leq M(1-z_1), a=1\\
g(x) \leq M(1-z_2), a=0\\
z_1 + z_2 &= 1
\end{align}
$$

### If a then  \\(f(x)= 0\\)

This is nothing but two implications
$$
\begin{align}
f(x)\leq M(1-q), a=1\\
-f(x) \leq M(1-q), a=0\\
\end{align}
$$

### If \\( f(x) \leq 0\\) then a

When \\(f(x)\\) becomes negative, a should be forced to be activated

$$
f(x)\geq -M(1-a)
$$

Notice that the case $f(x)=0$ is impossible to handle consistently in practice as solvers work finite precision and tolerances. If it is absolutely vital that the **a** is activated for $f(x)=0$, a margin has to be added, and this margin has to be selected ad-hoc so it is consistent with the numerical tolerance of the solver

$$
f(x)\geq \epsilon -M(1-a)
$$


### If \\( f(x) \leq 0\\) then a else not a

Bla bla

### If \\( f(x) \leq 0\\) then  \\( g(x) \leq 0\\)

Bla bla

### y = ab, a and b binary

Binary multiplication is nothing but logical and, hence

$$
y \leq a, y\leq b, y\geq a+b-1
$$

### y = ax, a binary x continuous (or integer)

Multiplication of binary and non-binary should be seen as a logical operation
$$
a  \rightarrow y = 0\\
1-a \rightarrow y = x\\
$$

This is implemented using our models for implications

$$
M(1-a) \leq y-x \leq M(1-a)\\
Ma \leq y \leq Ma
$$

Bla bla


### y = max(x)

The maximum of a vector can be though of a logical model **if x(1) larger than all other elements then y = x(1) elseif x(2) ...**. Hence, introduce a binary variable for every possibility, and 

$$
z_1 \rightarrow y = x_1, x_1 \geq x\\
\vdots\\
z_n \rightarrow y = x_n, x_n \geq x\\
$$

This is finalized with implications

$$
M(1-z_1) \leq y - x_1\leq M(1-z_1), x-x_1 \leq M(1-z_1)\\
\vdots\\
M(1-z_n) \leq y - x_1\leq M(1-z_n), x-x_n \leq M(1-z_n)\\
\sum_{i=1}^n z_1 = 1
$$


### y = min(x)

Simply use **min(x) = -max(-x)**


### y = sort(x)

Bla bla

