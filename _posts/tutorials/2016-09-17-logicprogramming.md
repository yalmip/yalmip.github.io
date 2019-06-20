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

Throughout the examples here, $a$ and $b$ represent scalar binary variables, $z$ represent a vector of binary variables, and $x$ represent continuous variables.


1. Try to represent things with disjoint events representing "exactly one thing of these occur"
2. Try to arrive at "binary variable implies set of constraints"
3. Introduce intermediate auxilliary variables to keep things clean
4. Decompose the logic using intermediate variables to connect parts


### s = NOT a

With binary \\(a = 1\\) representing true and \\(a = 0\\) representing false, logical negation turns into 

$$
s = 1-a
$$

In YALMIP, we either do this manually, or use the logical operator. Let's try it where we force \\(a\\) to be true, and then try to maximize \\(s\\) (only feasible solution is \\(s == 0\\))

````matlab
binvar s a
optimize([s == not(a), a == 1], -s);value(s)
optimize([s == 1 - a, a == 1], -s);value(s)
````

### s = a AND b

\\(s\\) has to be \\(1\\) if both \\(a\\) and \\(b\\) are 1. \\(s\\) has to be \\(0\\) if  either of \\(a\\) and \\(b\\) are 0.

$$
s \geq a + b -1,~s \leq a,~s\leq b
$$

The idea generalizes to an arbitrary number of binary variables \\(s = z_1 \& z_2 \& \ldots z_\n\\) with

$$
s \geq \sum_{i=1}^n z_i - (n-1), s\leq z
$$

Solve a trivial problem where the only feasible solution is \\(s = 0\\)
````matlab
binvar s a b
optimize([s == a & b, a == 1, b == 1], s);value(s)
optimize([s >= a + b - 1, s<=a, s<=b, a == 1, b == 1], -s);value(s)
````

### s = a OR b

\\(s\\) has to be \\(1\\) if  either of \\(a\\) and \\(b\\) are 1, and \\(0\\) if none of them are 1.

$$
s \geq a,~s\geq b, s \leq a + b 
$$

### s = a XOR b

\\(s\\) has to be \\(1\\) if  exactly one of \\(a\\) and \\(b\\) are 1, and \\(0\\) otherwise

$$
s \geq a - b, s \geq b-a, s \leq a + b, a + b\leq 2-s
$$

The idea generalizes to an arbitrary number of binary variables  with

$$
s \geq z_i - \sum_{j\ne i} z_j - (n-1), s\leq \sum_{i = 1}^n z_j \leq n + (1-n)s
$$


### If a then b

Bla bla

### If a then  \\(f(x)\leq 0\\)

Bla bla

### If a then  \\(f(x)\leq 0\\) else  \\(f(x)\geq 0\\)

Bla bla

### If a then  \\(f(x)\leq 0\\) else  \\(g(x)\leq 0\\)

Bla bla

### If \\( f(x) \leq 0\\) then a

Bla bla

### If \\( f(x) \leq 0\\) then a else not a

Bla bla

### If \\( f(x) \leq 0\\) then  \\( g(x) \leq 0\\)

Bla bla

### y = ab, a and b binary

Bla bla

### y = ax, a binary x continuous (or integer)

Bla bla


### y = max(x)

Bla bla


### y = min(x)

Bla bla


### y = sort(x)

Bla bla

