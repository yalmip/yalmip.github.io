---
layout: single
excerpt: "MATLAB no longer required! Recommended though."
title: Octave support in YALMIP
tags: [Octave]
comments: true
date: '2014-04-16'
---

I have been asked several times if I would consider an [Octave](http://www.gnu.org/software/octave/) port. My answer has been no, based on my, as it turns out, flawed idea that it would require a lot of changes (and my general lack of interest in Octave). Well, yet another request came, and I decided to download Octave and test it. Turned out the changes required were absolutely minimal. Hence, as of now, YALMIP runs in both MATLAB and Octave. 

I have only tested Octave 3.8.1 on Win64, and have no idea if it works in other settings.

Besides being slower, the main issue now in Octave from a YALMIP perspective is the lack of solvers. At this point, the only solvers tested with YALMIP are [GLPK](/solver/glpk) (shipped with the core of Octave) and [sdpt3](/command/sdpt3) (recently updated by Michael Grant to compile in Octave, (https://github.com/sqlp/sdpt3). Also [sedumi](/command/sedumi) is updated to compile in Octave (https://github.com/sqlp/sedumi), but I have not managed to compile it and test it on my machine.

There are most likely many small issues left to be updated. If you experience any problems, please report this.

To speed up YALMIP slightly in Octave, edit ismembcYALMIP.m and follow the instructions.
