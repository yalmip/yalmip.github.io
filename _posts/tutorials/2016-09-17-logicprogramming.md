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



### s = NOT a

With binary \\(a = 1\\) representing true and \\(a = 0\\) representing false, logical negation turns into 

$$
s = 1-a
$$

In YALMIP, we either do this manually, or use the logical operator. Let's try it where we force \\(a\\) to be true, and then try to maximize \\(s\\) (only feasible solution is \\(s == 0\\)
````matlab
binvar s a
optimize([s == not(a), a == 1], -s);value(s)
optimize([s == 1 - a, a == 1], -s);value(s)
````

### s = a AND b

\\(s\\) has to be \\(1\\) if both \\(a\\) and \\(b\\) are 1. Hence

$$
s \geq a + b -1 
$$
\\(s\\) has to be \\(0\\) if  either of \\(a\\) and \\(b\\) are 0.
$$
s \leq a, s\leq b
$$

The idea generalizes to an arbitrary number of binary variables \\(s = z_1 \& z_2 \& \ldots z_\n$ with

$$
s \sum_{i=1}^n z_i - (n-1), s\leq z
$$

Solve a trivial problem where the only feasible solution is \\(s = 0\\)
````matlab
binvar s a b
optimize([s == a & b, a == 1, b == 1], s);value(s)
optimize([s >= a + b - 1, s<=a, s<=b, a == 1, b == 1], -s);value(s)
````

### s = a OR b

Bla bla

````matlab
binvar a
````


### s = a XOR b

Bla bla

````matlab
binvar a
````

### If a then b

Bla bla

````matlab
binvar a
````

### If a then  \\(f(x)\leq 0\\)

Bla bla

````matlab
binvar a
````

### If a then  \\(f(x)\leq 0\\) else  \\(f(x)\geq 0\\)

Bla bla

````matlab
binvar a
````

### If a then  \\(f(x)\leq 0\\) else  \\(g(x)\leq 0\\)

Bla bla

````matlab
binvar a
````


### If \\( f(x) \leq 0\\) then a

Bla bla

````matlab
binvar a
````

### If \\( f(x) \leq 0\\) then a else not a

Bla bla

````matlab
binvar a
````

### y = ab, a and b binary

Bla bla

````matlab
binvar a
````

### y = ax, a binary x continuous (or integer)

Bla bla

````matlab
binvar a
````

### y = max(x)

Bla bla

````matlab
binvar a
````

### y = min(x)

Bla bla

````matlab
binvar a
````

### y = sort(x)

Bla bla

````matlab
binvar a
````
