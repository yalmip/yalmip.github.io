---
title: "Stochastic programming"
category: unclassified
level: 4
tags: [stochastic programming]
excerpt: "Probably working."
sidebar:
  nav: "tutorials"
published: false  
---

> This page is under development and is currently mainly used for internal development. 

## Background

Stochastic programming, a little known feature available in development branches for over a decade. Almost getting ready to actually release.

## Defining variables

The two driving principles in the setup of stochastic programming models is that random variables are introduced and manipulated like any other variables in YALMIP, and and a strive for notational consistency with the Statistics toolbox. Communicating properties of the variables are done via the [uncertain](/command/uncertain) command, thus making stochastic and deterministic worst-case models very similiar.

### Scalar Gaussian variables

Starting from the most basic example, we can define a scalar variable, with zero mean and unit standard deviation (thus unit variance). There is no change in the way we define a variable, and when we construct a model we communicate properties through trailing arguments in [uncertain](/command/uncertain). The arguments typically follow the parameters used in the command **random** in the Statistics toolbox. For a normal distribution, there are two properties, mean and standard deviation.

````matlab
sdpvar w
Model = [uncertain(w,'normal',0,1)];
````

Defining a model with non-zero mean and non-unit standard deviation is done accordingly.

````matlab
sdpvar w
w_mean = 2;
w_std = 5;
Model = [uncertain(w,'normal', w_mean, w_std)];
````

### Multivariate Gaussian variables

Multivariate Gaussian variables can be defined in several ways. For the case when we have a vector of independent Gaussian variables, we can simply use the **'normal'** distribution and supply vector means and standard deviations

````matlab
w = sdpvar(2,1)
w_mean = [2;2];
w_std = [1;5];
Model = [uncertain(w,'normal', w_mean, w_std)];
````

An alternative is to declare multiple scalar Gaussian variables.

````matlab
w1 = sdpvar(1,1);
w2 = sdpvar(1,1);
Model = [uncertain(w1,'normal', 2, 1), uncertain(w2,'normal',2,5)];
````

To support the general multivariate Gaussian case, the distribution name is **'mvnrnd'**. Important to remember is that this distribution is parameterized in the mean and **covariance**. Consequently, the variable defined in the previous example would be

````matlab
w = sdpvar(2,1);
w_mean = [2;2];
w_std = [1;5];
Model = [uncertain(w,'mvnrnd', w_mean, diag(w_std.^2))];
````

A special purpose distibution is **'mvnrndfactor'** which is defined through a square-root of the covariance instead. Hence, the following two models are equivalent

````matlab
w = sdpvar(2,1);
w_mean = [2;2];
R = [1 2;3 4];
S = R'*R;
Model1 = [uncertain(w,'mvnrnd', w_mean, S)];
Model2 = [uncertain(w,'mvnrndfactor', w_mean, R)];
````

In the examples so far, both the mean and the standard deviations, covariances and covariance factors have been constant. This is however not necessary, and we will later see models where some of these are decision variables, or uncertain variables, and the factor model turns out to be particularily useful in those cases motivating this special purpose parameterization. 

### Other standard distributions

Distributions supported in the Statistics toolbox are available (under development, very limited at the time). Hence, knowing the logic of parameters we can define, e.g., 

````matlab
w = sdpvar(2,1);

mu = [2;3];
Model = [uncertain(w,'exponential', mu)];

mu = [2;3];
shape = [4;5];
Model = [uncertain(w,'logistic', mu, shape)];

mu = [2;3];
theta = [4;5];
Model = [uncertain(w,'cauchy', mu, theta)];

L = [2;3];
U = [4;5];
Model = [uncertain(w,'uniform', L, U)];

L = [2;3];
U = [4;5];
Model = [uncertain(w,'t', L, U)];
````

### Data driven distributions

### Summary

| Distribution Name | Example Notation                                   | Parameter 1        | Parameter 2        | Parameter 3        | Description / Notes                                                         |
|-------------------|----------------------------------------------------|--------------------|--------------------|--------------------|-----------------------------------------------------------------------------|
| normal            | uncertain(w, 'normal', mean, std)                  | mean               | std                | —                  | Scalar or vector Gaussian                                                   |
| mvnrnd            | uncertain(w, 'mvnrnd', mean, covariance)           | mean               | covariance         | —                  | Multivariate Gaussian. Covariance is a matrix.                              |
| mvnrndfactor      | uncertain(w, 'mvnrndfactor', mean, R)              | mean               | R (covariance root)| —                  | Multivariate Gaussian via covariance root (R so that covariance = R'*R).    |
| exponential       | uncertain(w, 'exponential', mu)                    | mu                 |                    | —                  | Exponential distribution. Parameters follow MATLAB conventions.             |
| logistic          | uncertain(w, 'logistic', mu, s)                    | mu (location)      | s (scale)          | —                  | Logistic distribution. Location and scale.                                  |
| cauchy            | uncertain(w, 'cauchy', x0, gamma)                  | x0 (location)      | gamma (scale)      | —                  | Cauchy distribution. Location and scale.                                    |
| uniform           | uncertain(w, 'uniform', a, b)                      | a (lower)          | b (upper)          | —                  | Uniform distribution on [a, b].                                             |
| t                 | uncertain(w, 't', nu, mu, sigma)                   | nu (degrees)       | mu (location)      | sigma (scale)      | Student's t-distribution. Degrees of freedom, location, scale.              |

### Mixtures

# Setting up probabilistic constraints

## The classical Gaussian

### Exponential cone approximations

## General distributions

### Characteristic function approach

## Data-driven and ambiguous distributions

## Robust probabilistic constraints

