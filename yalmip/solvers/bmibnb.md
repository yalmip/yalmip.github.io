---
title: "BMIBNB"
layout: single
sidebar:
  nav: "solvers"
---

Built-in global solver for nonconvex problems.

### Availability

BMIBNB is shipped with YALMIP.

### YALMIP
BMIBNB is invoked by using `sdpsettings('solver','bmibnb')`

Most important options
 <table border="1" cellspacing="1" style="border-collapse: collapse" width="100%" bordercolor="#000000" bgcolor="#FFFFFF" id="table1">
        <tr>
          <td width="310"><code>sdpsettings('bmibnb.roottight',[0|1])</code></td>
          <td>Improve variable bounds at root-node by performing bound 
			strengthening based on the full relaxed model (Can be very expensive, 
			but lead to improved branching)</td>
        </tr>
        <tr>
          <td width="310"><code>sdpsettings('bmibnb.lpreduce',[0|1])</code></td>
          <td>Improve variable bounds in all nodes by performing bound 
			strengthening using only the scalar constraints (including scalar cut 
			constraints) in the model (Can be very expensive, but lead to 
			improved branching, in particular for semidefinite problems)</td>
        </tr>
        <tr>
          <td width="310"><code>sdpsettings('bmibnb.lowersolver', solvertag)</code></td>
          <td>Solver for relaxed problems.</td>
        </tr>
        <tr>
          <td width="310"><code>sdpsettings('bmibnb.uppersolver', solvertag)</code></td>
          <td>Local solver for computing upper bounds.</td>
        </tr>
        <tr>
          <td width="310"><code>sdpsettings('bmibnb.lpsolver', solvertag)</code></td>
          <td>LP solver for bound strengthening 
			(only used if <code>bmibnb.lpreduce</code> is set to 1)</td>
        </tr>
        <tr>
          <td width="310"><code>sdpsettings('bmibnb.numglobal', [int])</code></td>
          <td>A major computational burden along the branching process is to 
			solver the upper bound problems using a local nonlinear solver. By 
			setting this value to a finite value, the local solver will no 
			longer be used when the upper bound has been improved <code>numglobal</code> times. 
			This option is useful if you believe your local solver quickly gives 
			globally optimal solutions. It can also be useful if you have a 
			feasible solution and just want to compute a gap. In this case, use 
			the <code>usex0</code> option and set <code>numglobal</code> to 0.</td>
        </tr>
        </table>


### Comments
BMIBNB is an implementation of a standard [BMIBNBTheory branch & bound algorithm for nonconvex problems], based on linear programming relaxations and convex envelope approximations.

The solver relies on external linear and quadratic programming solvers for solving the lower bounding relaxation problems, and nonlinear solvers for the upper bound computations.

Do not expect too much. Global solutions are extremely hard to compute, and this is a fairly simple implementation. Problems with more than 10 variables is often beyond the capabilities of this solver, although you might be lucky. 

### See also
[Global optimization tutorial](/yalmip/tutorials/globaloptimization)
