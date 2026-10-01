# PSI Numerical Methods 2026 - Homework

## A higher-order symplectic integrator

In this assignment, you will implement and demonstrate the properties
of an ODE integrator that combines some of the best features of the
Runge-Kutta 4 integrator and the Semi-implicit Euler symplectic
integrator.  Specifically, we will implement the "Fourth-Order
Yoshida" integrator and investigate how it works for integrating
gravitational orbits (the Kepler problem).

### Setup

In Tutorial 3, you implemented the Semi-implicit Euler method, which
first finds the slope in the velocity at the starting point,

```math
v_{n+1} = v_n + g(t_n, x_n, v_n) h
```

and then uses that *new* velocity to find the slope in the position:

```math
x_{n+1} = x_n + f(t_n, x_n, v_{n+1}) h
```

Where the functions `f` and `g` compute the derivatives in *v* and *x*:

```math
g(t_n, x_n, v_n) = \frac{dv}{dt}
```

and

```math
f(t_n, x_n, v_n) = \frac{dx}{dt}
```


Your implementation might have been a bit messy because our functions
were set up to solve general ODEs, and we were packing the positions
and velocities into one long vector, but in the Semi-implicit Euler
step, we had to handle the positions and velocities separately, so we
had to un-pack and re-pack those vectors.

So before we start, let's "refactor" our code to not "pack" the
positions and velocities into one vector, but instead keep them
separate.  And let's also switch over to using the language of
Hamiltonian mechanics.

Specifically, instead of creating an `f(t, x)` function that returns
the time derivative of the general vector `x`, we will create *two*
functions, which we'll call `dHdq` and `dHdp`, the partial derivatives
of the Hamiltonian, with respect to position `q` and momentum `p`.

In Hamiltonian mechanics, we have

```math
\dot{q} = \frac{\partial H}{\partial p}
```

and

```math
\dot{p} = -\frac{\partial H}{\partial q}
```

so where before we set up `f(t, x, **f_kwargs)`, now we will have two
functions,
```python
def dHdq(t, q, p, **dH_kwargs):
    # ...
    return dhdq

def dHdp(t, q, p, **dH_kwargs):
    # ...
    return dhdp
```

I rewrote our `evolve()`, `step_forward_euler()`, `step_midpoint()`,
and `step_rk4()` functions, in the notebook that you'll find in the
same directory as this document.  Please feel free to use that
notebook as the starting point for your assignment.

You will have to write the `dHdp` and `dHdq` functions for the
Newtonian gravity (Solar system orbit) problem, like the function we
called `f_newton_onebody()`.  The Hamiltonian for this problem is

```math
H(q, p) = \frac{p^2}{2 m} - \frac{G M m}{\lVert q \rVert}
```

where we will use `q` for position and `p` for momentum; so your
`dHdp` function should look a lot like the derivative-of-position part
of `f_newton_onebody()` and the `dHdq` function should look a lot like
the derivative-of-velocity part of `f_newton_onebody()`.  For the
Newtonian gravity problem, both these functions should return a
length-3 vector.

### The Yoshida integrator

I am using the
[Wikipedia page](https://en.wikipedia.org/wiki/Leapfrog_integration#4th_order_Yoshida_integrator)
about the 4th-order Yoshida integrator as a reference.

This is a bit similar to the Runge-Kutta 4 algorithm, in that we take
different steps between the initial and final time points, and then
combine those values to get our final estimate.

As written in the Wikipedia article, the Yoshida algorithm produces a
*series* of points, where each one *updates* the previous one -- so
you will step from `x_1` to `x_2`.

Let's start by rewriting the Wikipedia algorithm in terms of the
Hamiltonian: their `x` variables become our `q`, their `v` become our
`p`, and when we're updating `q`, instead of just using the `v`
values, we're going to call our Hamiltonian `dHdp` function; their
acceleration `a` is our `-dHdq` function:

```math
\begin{eqnarray}
dq   &=& \frac{\partial H}{\partial p} |_{p_i} \\
q_1  &=& q_i + c_1 dq h \\
dp   &=& -\frac{\partial H}{\partial q} |_{q_1} \\
p_1  &=& p_i + d_1 dp h \\
dq_1 &=& \frac{\partial H}{\partial p} |_{p_1} \\
q_2  &=& q_1 + c_2 dq_1 h \\
...  & & \\
p_3  &=& ... \\
q_4  &=& ... \\
\end{eqnarray}
```

The first part of my implementation of the Yoshida algorithm looks
like this:

```python
def step_yoshida(t, q, p, h, dHdp, dHdq, dH_kwargs):

    # compute Yoshida coefficients
    x0 = -2**(1/3) / (2 - 2**(1/3))
    x1 = 1 / (2 - 2**(1/3))
    c1 = c4 = 0.5 * x1
    c2 = c3 = 0.5 * (x0 + x1)
    d1 = d3 = x1
    d2 = x0

    # First Yoshida step: first call dHdp to get the "q-dot" derivative
    dq1 =  dHdp(t, q, p, **dH_kwargs)
    # update the position
    q1 = q + dq1 * c1 * h
    t1 = t + c1 * h
    # call dHdq to get the (negative) "p-dot" derivative
    dp1 = -dHdq(t1, q1, p, **dH_kwargs)
    p1 = p + d1 * dp1 * h
    # ....
    return q4, p3
```

### Your tasks:

* write the `dHdp_kepler(t, q, p, GM=1., m=1.)` and `dHdq_kepler(t, q,
  p, GM=1., m=1.)` functions for the Newtonian gravity problem.

* write the `step_yoshida()` algorithm.

* test your algorithm for its short-term error properties.  How does
  it compare to the Semi-implicit Euler and RK4 algorithms for its
  error properties?

* test your algorithm for its long-term stability properties.  How
  does it compare with the non-symplectic algorithms and with the
  Semi-implicit Euler?

* test your algorithm for total energy conservation.  (The Hamiltonian
  is equal to the kinetic plus potential energy here.)  How does its
  energy conservation compare against the other algorithms, if you run
  it for a long time?

For each of these questions, please make a plot and write a few lines
of text describing your interpretation.

(Hint: if your algorithm is working right, it should give you both
good convergence (good errors) and also have good stability
(symplectic)!)
