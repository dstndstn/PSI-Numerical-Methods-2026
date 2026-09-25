# PSI Numerical Methods 2026 -- Tutorial 3

Today we're going to look at solving ODEs, continuing on from what we
did in lecture.

In this directory you can find the notebook I created during the
lecture, cleaned up a bit, and with a plot about the error performance
of Forward Euler, Midpoint, and Runge-Kutta 4.  Go ahead and copy that
notebook as the starting point for the tutorial today, if you like.

https://github.com/dstndstn/PSI-Numerical-Methods-2026/tree/main/Tutorial-3

In a Symmetry Jupyterhub notebook, you should be able to run this command
to copy the `lecture.ipynb` file from my home directory to your current directory, by running this in a notebook cell:

```
! cp --no-clobber /home/dlang/lecture.ipynb tutorial3.ipynb
```

Let's look at the Newtonian gravity problem.  It looks like RK4 is doing a pretty good
job on our example problem, but if we zoom in, we can see that the orbits are not closed -
RK4 is not preserving the energy in our problem.

## Task: long-term stability of RK4

Try running RK4 for a long time with the same step size -- eg, in our
setup from class, try running it up to `t1 = 4200` with `N = 42000`.
Try pulling out the position and velocity values from the vector `x`
of results, and compute the kinetic energy and the gravitational
potential energy.  Is the sum of kinetic and gravitational potential
energy constant?

## Semi-implicit Euler

We're going to make a remarkably simple change to the Forward Euler
method, resulting in a step function with much improved stability
properties.

The velocity is the same as in Forward Euler:

```math
v_{n+1} = v_n + g(t_n, x_n, v_n) h
```

But for the position, we're going to use the *new* velocity:

```math
x_{n+1} = x_n + f(t_n, x_n, v_{n+1}) h
```

Where the functions `f` and `g` compute the derivatives:

```math
g(t_n, x_n, v_n) = \frac{dv}{dt}
```

and

```math
f(t_n, x_n, v_n) = \frac{dx}{dt}
```

Now, in our code, we have packed the position and velocity into a
single `x` vector, so we don't really have access to the `f` and `g`
parts separately.  But if we call the `f` function as we have defined
it, we get *both* the `dx/dt` and `dv/dt` parts; we can just ignore
the part we don't need.

## Task:

Write the semi-implicit Euler method.  This might be a bit messy
because of the packing and unpacking of position and velocity
components in our `x` vector and the `x_next` result.

Once you've got that working, test it for the same Newton problem that
we ran RK4 on above.  You should find that the results are *stable*
over long periods -- the planet doesn't shoot off into the outer Solar
system, even if you run it for a long time.

Pull out the position and velocity, as before, and compute and plot
the kinetic, potential, and total energy.

Is the energy constant?

Does the simulation look stable?

Also, observe that Semi-implicit Euler still has much worse error
properties than RK4, until things go off the rails for RK4.  In the
notebook, I added a plot about the error performance of the different
step functions, on the Simple Harmonic Oscillator problem.  Try adding
your Semi-implicit Euler to that.  How does it compare?

## Symplectic integrators

I may have mis-spoken in class and said that this method *preserves
energy*.  I was, ahem, *not quite right* about that, as you just saw.
Instead, it preserves a slightly more complicated thing: *phase-space
volume*.

This is a bit easier to see mathematically in the Simple Harmonic
Oscillator (SHO) than in the gravity problem.

In SHO, the semi-implicit Euler update equations look like this:

```
v_next = v_n - w**2 * x_n * h
x_next = x_n + v_next * h
# Or,
x_next = x_n + (v_n - w**2 * x_n * h) * h
```

You can write this as a matrix equation,

```
[ x_next ]  =  [ 1 - w**2 * h**2      h ] [ x_n ]
[ v_next ]     [   - w**2 * h         1 ] [ v_n ]
```

and the *determinant* of that matrix is 1.  This means that if you
look at a little region in `[x, v]` space around a given point and run
the semi-implicit Euler update on it, that turns into a *new* little
region in `[x,v]` space, where that region has *the same volume*.

The math and physics here get deep pretty quickly, but if you write
your system in terms of the Hamiltonian, eg, with *position* and
*momentum* as your variables, then there are many beautiful
connections.  An algorithm that preserves *phase space volume* is
called *symplectic*.  It *turns out* that a symplectic algorithm is
*exactly solving* a *nearby Hamiltonian*.

The semi-implicit Euler step is a *symplectic* algorithm!

## Extension Tasks:

In Semi-implicit Euler, we kind of arbitrarily decided to step in `v`
first, and then `x`.  We could do it the other way around.  This would
result in somewhat different behavior in detail but the same overall
performance.  But what if we *alternated* those two steps?

That step function is called the Leapfrog method.

Try implementing it!  Measure its performance in terms of stability,
accuracy (error), and conservation of energy.

