# PSI Numerical Methods 2026 - Tutorial 2

Today we're going to look at some Numpy basics, timing code, and Matplotlib plots.

## Numpy arrays

Numpy arrays have one very important property: their sizes are constant.  Unlike a built-in Python list, it's not easy to grow a Numpy array.
As a result, we often have to take a bit of care when creating or initializing Numpy arrays.

We are going to try four different ways to create a Numpy array containing Gaussian random values, and measure how long they take to run.

Here are the four approaches:

1. Create a Python list, convert it to a Numpy array.

```
def list_rand(N):
    r = []
    for i in range(N):
        r.append(np.random.normal())
    return r
```

2. Create a Numpy array, and append each random number to it.

```
def append_rand(N):
    r = np.array([])
    for i in range(N):
        r = np.append(r, np.random.normal())
    return r
```

3. Pre-allocate a Numpy array, and fill in each item.

```
def alloc_rand(N):
    r = np.zeros(N)
    for i in range(N):
        r[i] = np.random.normal()
    return r
```

4. Use the built-in ability to create an array in the first place.

```
def numpy_rand(N):
    r = np.random.normal(size=N)
    return r
```

To begin, look at each function and make your own prediction of which
one is going to be fastest.  This is a good practice in scientific
research --- before you make a plot, **predict** what is going to look
like!  This forces you to engage with your current ideas of the
problem, and think about what you are trying to demonstrate with the
plot.  Then, if you are **surprised** by the result, you should pay
very careful attention, because either you made a mistake in creating
the plot, or you just **learned something**!

In the lecture, I showed you the `%%timeit` magic in Jupyter
notebooks, which runs the contents of the current notebook cell
several times and prints out how long it took.  But we want to
_record_ how long things take, so we'll do the timing ourselves.  One
way to do that is this:

```
import time
def time_it(func, arg):
    t0 = time.monotonic()
    result = func(arg)
    t1 = time.monotonic()
    return t1-t0, result
```

and then we can use it like this:

```
N = 10000
t,result = time_it(list_rand, N)
```

Notice that we're passing _the function_ and _an argument_ to the
`time_it` function, and then `time_it` is actually calling the
function.

### Tasks:

Create a new Jupyter notebook and copy-n-paste in the code snippets
above.  You'll also need to put at the top of your notebook some imports:

```
import numpy as np
import time
import matplotlib.pyplot as plt
```

1. Predict which of the functions above (`list_rand`, `append_rand`, `alloc_rand`, or `numpy_rand` is going to be fastest.
2. Time how long it takes to run each of the four functions above, for different `N` values
up to, say, `100000`.
3. Plot how long each function takes with respect to `N`.  In your plot, put labels on the
axes, given the plot a title, and create a legend.
4. Think about what the plot is showing.  Does anything surprise you?

## Numpy indexing, and plots

One of the great features of numpy is that if you have a numpy array,
you can perform operations on every element in that array without
having to explicitly loop through the array.  This _vectorization_ can
lead to fast, simple-looking code.

Another nice feature is that you can pull out a _subset_ of an array
by indexing it with a _boolean_ array; the result will be an array
containing only the elements where the index is `True`.  For example,

```
x = np.arange(9)
print('x:', x)
near_four = (np.abs(x - 4) <= 1)
print('near_four:', near_four)
print('x[near_four]:', x[near_four])
```

will print:

```
x: [0 1 2 3 4 5 6 7 8]
near_four: [False False False  True  True  True False False False]
x[near_four]: [3 4 5]
```

That is: `x[near_four]` is pulling out only the elements in `x` where
`near_four` is `True`.  Note that `x` and `near_four` need to have the
_same shape_ for this to work.

Another way of indexing numpy array is to create an array of
_integers_ giving the _indices_ into the array that you want to pull
out.  For example, here I'm creating array `x` containing the squared
values, and then indexing it with an integer array:

```
x = np.arange(9)**2
print('x:', x)
some_indices = np.array([3, 4, 5])
print('some_indices:', some_indices)
print('x[some_indices]:', x[some_indices])
```

This will print:

```
x: [ 0  1  4  9 16 25 36 49 64]
some_indices: [3 4 5]
x[some_indices]: [ 9 16 25]
```

Note that `some_indices` is _not_ the same shape as `x`; but the
resulting `x[some_indices]` array will have the same length as
`some_indices`.

### Tasks:

1. Create two arrays, `x` and `y`, containing `100000` random points each,
and plot them as dots in a 2-d plot.
2. Use boolean indexing like given above to plot just the `x` and `y` points where
`y > x`, and the point `(x,y)` has distance greater than 3 from the origin.
3. Make a 2-d histogram of the `x` and `y` points.

## Numpy broadcasting and plotting images

Another feature of numpy is called _broadcasting_.  If you have two
numpy arrays, you can use _broadcasting_ to tell numpy to combine the
array elements in different ways.

Imagine you have a function that operates on 2 variables - maybe it is
evaluating a 2-d scalar field like the gravitational potential - and
you want to evaluate that function on a grid and show the result.
While you can use Matplotlib's `scatter()` function to plot points
that each have a different color, Matplotlib also has special
functions for dealing with _images_, which are represented as 2-d
arrays.

For example, let's say the function we want to examine is:

```
def my_potential(x, y):
    return np.exp(np.sin(x) * np.sin(2*y))
```

and we want to examine it on a from grid on `x,y` from zero to `2 pi`.

First, let's create `xvals` and `yvals` arrays (1-d arrays) where we
want to evaluate the function.  To catch some kinds of bugs, let's
make them _not_ the same size:

```
xvals = np.linspace(0, 2.*np.pi, 200)
yvals = np.linspace(0, 2.*np.pi, 201)
```

If we try to call `my_potential(xvals, yvals)`, that's going to fail,
because numpy will see that these are both 1-d arrays, so it will try
to do _element-wise_ operations, but it will find that `xvals` and
`yvals` are different sizes, and it will fail.

Instead, we need to tell numpy that we want to operate on _all
combinations_ of `xvals` and `yvals` in a grid.

One option is to do this explicitly, using, eg, the `meshgrid` function:

```
xgrid,ygrid = np.meshgrid(xvals, yvals)
pgrid = my_potential(xgrid, ygrid)
```

Another option is to use _fancy indexing_ to tell numpy that you want
to treat `xvals` and `yvals` as 2-d arrays.  You can use the special
symbol `np.newaxis` to tell numpy that you want it treat your 1-d
array as though it had 2 dimensions.  For example,

```
yvals[:, np.newaxis].shape
```

should show `(201,1)` -- numpy is treating it like a 2-d array.  The
fancy thing is that that `np.newaxis` dimension will _broadcast_ to
the size needed to operate with another array: if you write

```
yvals[:, np.newaxis] * np.array([2,3])
```

it will _pretend_ that `yvals` is a `200 x 2` array that can be
multiplied by a length-2 array.  That's _broadcasting_.  The `:`
index means "all elements in this dimension of the array".

We can do that with two 1-d arrays like this:

```
pvals = my_potential(xvals[np.newaxis,:], yvals[:,np.newaxis])
```

and then `pvals` will be a 2-d array.

### Tasks

1. Describe what `meshgrid` does.  If we run this,

```
my_x = np.array([10,20,30])
my_y = np.array([4,5])
mesh_x, mesh_y = np.meshgrid(my_x, my_y)
```

then what are the _shapes_ and _values_ in `mesh_x` and `mesh_y`?

2. Experiment with fancy indexing and broadcasting a bit.  What do these commands produce?  What are the shapes and contents of the results?

```
my_x[:,np.newaxis] + my_y[np.newaxis,:]
```

```
my_x[np.newaxis,:] + my_y[:,np.newaxis]
```

3. Copy-n-paste in the code snippets above.  What are the _shapes_ of
`xvals`, `yvals`, `xgrid`, `ygrid`, `pgrid`, and `pvals`?
Approximately how much memory do these variables use?

4. Modify your `time_func` from above to handle two arguments, and try
timing these two versions.  Is one of them faster than the other?  You
may need to increase the array sizes to be able to measure the
differences.

5. Notice how I arranged the `:` and `np.newaxis` indices for the
`xvals` and `yvals` arrays.  Why did I choose that order?  What shape
is the output?  What happens if you switch the orders?

6. Try two different ways of plotting these 2-d arrays:

```
fig,ax = plt.subplots()
xgrid,ygrid = np.meshgrid(xvals, yvals)
pgrid = my_potential(xgrid, ygrid)
s = ax.scatter(xgrid, ygrid, c=pgrid)
fig.colorbar(s)
```

```
fig,ax = plt.subplots()
pvals = my_potential(xvals[np.newaxis,:], yvals[:,np.newaxis])
im = ax.imshow(pvals)
fig.colorbar(im)
```

What happens if you try to increase the sizes of `xgrid` and `ygrid`
to, say, 1000?

7. Notice that the `imshow`---which tell matplotlib to treat the array as
an image---plots it with `y=0` at the _top_ of the plot.  Look at the
`imshow` documentation and see if you can figure out how to make it
put `y=0` at the bottom of the plot.  Which axes of the array is
matplotlib plotting on the horizontal and vertical?  (It may help to
`imshow` an array with very different sizes -- eg,
`ax.imshow(np.zeros((10, 20))`.

8. Finally, notice that `imshow` is setting the axis ranges to the _size_
of the `pvals` array, but we would really like the axis tick marks to
correspond to the values in `xvals` and `yvals`.  See if you can find
in the `imshow` documentation how to set the _extent_ of the plot.

