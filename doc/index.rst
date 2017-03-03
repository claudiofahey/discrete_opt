
Discrete Optimization (:mod:`discrete_opt.optimization`)
========================================================

.. currentmodule:: discrete_opt.optimization

.. toctree::
   :maxdepth: 2
   :caption: Contents:

   api_docs


Introduction
------------

The excellent `SciPy Optimization library <https://docs.scipy.org/doc/scipy/reference/tutorial/optimize.html>`_
provides a variety of methods for optimizing functions of real continuous variables.
However, these functions are not very useful when trying to optimize scalar functions of discrete integral variables.
This package provides methods for minimizing scalar functions of one or more discrete integral variables.
The domain of each variable is an arbitrary range of consecutive integers such as -1, 0, 1, 2, 3.

Of course, with functions of discrete variables, derivatives of not used.
Further, many systems with discrete variables have zero derivatives (i.e., flat spots)
at locations that are not minimums.
Consider the function :math:`f(x) = \text{floor}(0.01 * x^2)`.
This function has a value of 1 at x=10, 11, 12, 13, and 14.
The "derivative" at x=11 is 0 but that is not the local minimum.

The minimization functions in this package can be used by the :func:`scipy.optimize.minimize`
and :func:`scipy.optimize.minimize_scalar` functions by passing them in the `method` parameter.


Scalar Discrete Minimization (:func:`scalar_discrete_gap_filling_minimizer`)
----------------------------------------------------------------------------

This is the core function in this package.
It minimizes a real-valued scalar function of a scalar integer.

The function may have flat spots where f(a) == f(b) for a != b and this method will
attempt to search around and within the flat spots.
The function is assumed to have exactly one local minimum in the bracket.

It must be supplied with an initial bracket that surrounds or includes the local minimum.
Optionally, a starting point can be supplied as the middle entry of a 3-tuple bracket.
If a starting point is not supplied, then the integer closest to the mean of the end points is used.
Alternatively, the Golden Section method can be used to pick the starting point.

This method maintains a left and right bracket, where the function value is greater than the best known minimum.
It also maintains a list of best x values, and the function values at all of these x values equals the best known
minimum.

At each iteration, if there are only 3 points in the bracket (left, best, right), then a parabola is fit
and the minimum of the parabola (rounded to the nearest integer) becomes the next point to evaluate the function at.
This is identical to existing real-valued scalar minimization methods.

However, if there are 2 or more best points (e.g. the bracket is [0, 12, 13, 15] in the example above), then
a different approach is used. Each set of consecutive points, including the left bracket, best points, and
the right bracket, have a gap size that is simply the absolute value of the difference in their x values.
The largest gap is selected. Then an integer within the selected gap is chosen using
either the Golden Section method (the a or b side will be selected at random) or
the mean, again rounded to the nearest integer.

This iteration is repeated until the largest gap is 1, a maximum number of iterations has been
reached, or a maximum number of function evaluations has been reached.


Multivariate Discrete Minimization (:func:`multivariate_discrete_gap_filling_minimizer`)
----------------------------------------------------------------------------------------

For minimizing a real-valued scalar function of multiple integers, this function is available.

This function repeatedly uses the above mentioned scalar minimizer along arbitrary axes
of the multivariate function.
For example, in the first iteration, it will fix all
variables except for the first and the perform a discrete scalar optimization along
the first variable.
Then it will fix all variables except for the second and perform
a discrete scalar optimization along the second variable.
This is repeated for all variables (dimensions) and for a fixed number of iterations.
At this time, there is no other stopping condition.

If the `axes` parameter is supplied, the axes along which scalar minimization is performed can
be customized. The `axes` parameter is a list of axes. Each element represents an axis.
An axis of [1, 0, 0] means that the first variable will incremented by integral multiples of 1.
An axis of [1, 1, 0] means that both the first variable and the second variable
will be incremented by integral multiples of 1.
The `axes` parameter can also be used to customize the order in which variables are optimized.
If not specified, the `axes` parameter is effectively the identity matrix.


Exhaustive Discrete Minimization (:func:`simple_global_minimizer_spark`)
------------------------------------------------------------------------

One nice property of functions of bounded discrete variables is that all function evaluations
can be enumerated in a finite list. When the domain is small enough, all possible
inputs can be evaluated in order to identify the global minimum with absolute certainty.
This is sometimes useful when testing the above local minimizers which are generally
much faster but may not always converge to the global or even local minimum.

This function can be called
in the same way as :func:`multivariate_discrete_gap_filling_minimizer`.
It uses Apache Spark to parallelize the computation.


Utility Classes and Functions
-----------------------------

Some discrete functions may be very expensive to compute. The class
:class:`discrete_opt.util.CachedFunction` is
a function wrapper that caches function calls with the same arguments.
It will also record the number of cache hits and cache misses. It can optionally record
a history of cache misses (i.e. function evaluations).

The class :class:`discrete_opt.util.LoggingFunction` is a function wrapper that logs all function calls.

Refer to the full API documentation for other useful utilities.


Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`

