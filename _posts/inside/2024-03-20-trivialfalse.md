---
category: inside
subcategory: 2
permalink: triviallyfalse
excerpt: "Asking for the impossible"
title: Constraints evaluates to trivially false
tags: [Common mistakes]
date: '2009-08-29'
---

Sometimes when working with complex models, you might see an error about trivially false constraints when constructing the model

````matlab
x = sdpvar(5,1);
Model = [];
for i = 1:5
    Model = [Model, x(i)^2 <= 1];
    Model = [Model, i*x(i) + i<= 5*x(5) + 4];
    Model = [Model, x(i) >= 0];
end
Model
Error using  <=  (line 7)
Error using constraint (line 57)
Inequality constraint evaluated to trivial false (no decision variable in constraint)
 
````

Without thinking of it, you have added a constraint of the type 1<=0 or 0==1 etc. To debug this, unless you immediately know what line caused it, simply comment out code untill it disappears, and then you know where it is

This still fails, hence that line is not the problem

````matlab
Model = [];
for i = 1:5
    % Model = [Model, x(i)^2 <= 1];
    Model = [Model, i*x(i) + i<= 5*x(5) + 4];
    Model = [Model, x(i) >= 0];
end
Model
Error using  <=  (line 7)
Error using constraint (line 57)
Inequality constraint evaluated to trivial false (no decision variable in constraint)
````

This runs, hence we know the problematic line.

````matlab
Model = [];
for i = 1:5
    % Model = [Model, x(i)^2 <= 1];
    % Model = [Model, i*x(i) + i<= 5*x(5) + 4];
    Model = [Model, x(i) >= 0];
end
Model
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
|   ID|                    Constraint|   Coefficient range|
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
|   #1|   Element-wise inequality 1x1|              1 to 1|
|   #2|   Element-wise inequality 1x1|              1 to 1|
|   #3|   Element-wise inequality 1x1|              1 to 1|
|   #4|   Element-wise inequality 1x1|              1 to 1|
|   #5|   Element-wise inequality 1x1|              1 to 1|
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
````

When does it happen?

````matlab
Model = [];
for i = 1:5
    i
    Model = [Model, i*x(i) + i<= 5*x(5) + 4];
end

i =

     1


i =

     2


i =

     3


i =

     4


i =

     5

Error using  <=  (line 7)
Error using constraint (line 57)
Inequality constraint evaluated to trivial false (no decision variable in constraint)

```

Ok, so something bad happens when i equals 5, which is pretty easy to see. Confirm

````matlab
Model = [];
for i = 1:5
    if i == 5
       5*x(5) + 4 - (i*x(i) + i)
    end
    Model = [Model, i*x(i) + i<= 5*x(5) + 4];
end

ans =

    -1

Error using  <=  (line 7)
Error using constraint (line 57)
Inequality constraint evaluated to trivial false (no decision variable in constraint)
 
````

Yep, -1 ain't larger than 0.
