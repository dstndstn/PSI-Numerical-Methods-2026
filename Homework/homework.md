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


