---
layout: single
excerpt: "Name your constraints for easy reference"
title: Tagging constraints
tags:
comments: true
date: '2009-08-29'
---

When a list of constraint is generated, it is sometimes hard to remember which constraints model what. To alleviate this problem, you can tag your list of constraints easily with descriptions.

The syntax is very easy, simply add a : followed by a string.

````matlab
sdpvar x
Constraints = [x>=0]:'Positivity'
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
|   ID|      Constraint|               Type|          Tag|
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
|   #1|   Numeric value|   Element-wise 1x1|   Positivity|
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
````

This tagging can be used also inside a list of constraints, but due to the order of evaluation in MATLAB, it is important to add parentheses around the constraints.

````matlab
sdpvar x
Constraints = [(x>=0):'Positivity',(x=<1):'Bounded']
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
|   ID|      Constraint|               Type|          Tag|
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
|   #1|   Numeric value|   Element-wise 1x1|   Positivity|
|   #2|   Numeric value|   Element-wise 1x1|      Bounded|
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
````

Alternatively, use brackets.

````matlab
sdpvar x
Constraints = [[x>=0]:'Positivity', [x<=1]:'Bounded']
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
|   ID|      Constraint|               Type|          Tag|
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
|   #1|   Numeric value|   Element-wise 1x1|   Positivity|
|   #2|   Numeric value|   Element-wise 1x1|      Bounded|
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
````

The tag is useful not only for displays, but can also be used as an index.

````matlab
 Constraints('Bounded')
+++++++++++++++++++++++++++++++++++++++++++++++++++++++
|   ID|      Constraint|               Type|       Tag|
+++++++++++++++++++++++++++++++++++++++++++++++++++++++
|   #1|   Numeric value|   Element-wise 1x1|   Bounded|
+++++++++++++++++++++++++++++++++++++++++++++++++++++++
````
