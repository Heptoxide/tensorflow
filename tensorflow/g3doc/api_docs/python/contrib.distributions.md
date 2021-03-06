<!-- This file is machine generated: DO NOT EDIT! -->

# Statistical distributions (contrib)
[TOC]

Classes representing statistical distributions and ops for working with them.

## Classes for statistical distributions.

Classes that represent batches of statistical distributions.  Each class is
initialized with parameters that define the distributions.

### Base classes

- - -

### `class tf.contrib.distributions.Distribution` {#Distribution}

A generic probability distribution base class.

`Distribution` is a base class for constructing and organizing properties
(e.g., mean, variance) of random variables (e.g, Bernoulli, Gaussian).

### Subclassing

Subclasess are expected to implement a leading-underscore version of the
same-named function.  The argument signature should be identical except for
the omission of `name="..."`.  For example, to enable `log_prob(value,
name="log_prob")` a subclass should implement `_log_prob(value)`.

Subclasses can rewrite/append to public-level docstrings. For example,

```python
Subclass.prob.__func__.__doc__ += "Some other details."
```

would add the string "Some other details." to the `prob` function docstring.

### Broadcasting, batching, and shapes

All distributions support batches of independent distributions of that type.
The batch shape is determined by broadcasting together the parameters.

The shape of arguments to `__init__`, `cdf`, `log_cdf`, `prob`, and
`log_prob` reflect this broadcasting, as does the return value of `sample` and
`sample_n`.

`sample_n_shape = (n,) + batch_shape + event_shape`, where `sample_n_shape` is
the shape of the `Tensor` returned from `sample_n`, `n` is the number of
samples, `batch_shape` defines how many independent distributions there are,
and `event_shape` defines the shape of samples from each of those independent
distributions. Samples are independent along the `batch_shape` dimensions, but
not necessarily so along the `event_shape` dimensions (dependending on the
particulars of the underlying distribution).

Using the `Uniform` distribution as an example:

```python
minval = 3.0
maxval = [[4.0, 6.0],
          [10.0, 12.0]]

# Broadcasting:
# This instance represents 4 Uniform distributions. Each has a lower bound at
# 3.0 as the `minval` parameter was broadcasted to match `maxval`'s shape.
u = Uniform(minval, maxval)

# `event_shape` is `TensorShape([])`.
event_shape = u.get_event_shape()
# `event_shape_t` is a `Tensor` which will evaluate to [].
event_shape_t = u.event_shape

# Sampling returns a sample per distribution.  `samples` has shape
# (5, 2, 2), which is (n,) + batch_shape + event_shape, where n=5,
# batch_shape=(2, 2), and event_shape=().
samples = u.sample_n(5)

# The broadcasting holds across methods. Here we use `cdf` as an example. The
# same holds for `log_cdf` and the likelihood functions.

# `cum_prob` has shape (2, 2) as the `value` argument was broadcasted to the
# shape of the `Uniform` instance.
cum_prob_broadcast = u.cdf(4.0)

# `cum_prob`'s shape is (2, 2), one per distribution. No broadcasting
# occurred.
cum_prob_per_dist = u.cdf([[4.0, 5.0],
                           [6.0, 7.0]])

# INVALID as the `value` argument is not broadcastable to the distribution's
# shape.
cum_prob_invalid = u.cdf([4.0, 5.0, 6.0])

### Parameter values leading to undefined statistics or distributions.

Some distributions do not have well-defined statistics for all initialization
parameter values.  For example, the beta distribution is parameterized by
positive real numbers `a` and `b`, and does not have well-defined mode if
`a < 1` or `b < 1`.

The user is given the option of raising an exception or returning `NaN`.

```python
a = tf.exp(tf.matmul(logits, weights_a))
b = tf.exp(tf.matmul(logits, weights_b))

# Will raise exception if ANY batch member has a < 1 or b < 1.
dist = distributions.beta(a, b, allow_nan_stats=False)  # default is False
mode = dist.mode().eval()

# Will return NaN for batch members with either a < 1 or b < 1.
dist = distributions.beta(a, b, allow_nan_stats=True)
mode = dist.mode().eval()
```

In all cases, an exception is raised if *invalid* parameters are passed, e.g.

```python
# Will raise an exception if any Op is run.
negative_a = -1.0 * a  # beta distribution by definition has a > 0.
dist = distributions.beta(negative_a, b, allow_nan_stats=True)
dist.mean().eval()
```
- - -

#### `tf.contrib.distributions.Distribution.__init__(dtype=None, parameters=None, is_continuous=True, is_reparameterized=False, validate_args=True, allow_nan_stats=False, name=None)` {#Distribution.__init__}

Constructs the `Distribution`.

##### Args:


*  <b>`dtype`</b>: The type of the event samples. `None` implies no type-enforcement.
*  <b>`parameters`</b>: Python dictionary of parameters used by this `Distribution`.
*  <b>`is_continuous`</b>: Python boolean, default `True`. If `True` this
    `Distribution` is continuous over its supported domain.
*  <b>`is_reparameterized`</b>: Python boolean, default `False`. If `True` this
    `Distribution` can be reparameterized in terms of some standard
    distribution with a function whose Jacobian is constant for the support
    of the standard distribution.
*  <b>`validate_args`</b>: Whether to validate input with asserts. If `validate_args`
    is `False`, and the inputs are invalid, correct behavior is not
    guaranteed.
*  <b>`allow_nan_stats`</b>: Python boolean, default `False`. If `False`, raise an
    exception if a statistic (e.g., mean, mode) is undefined for any batch
    member. If True, batch members with valid parameters leading to
    undefined statistics will return `NaN` for this statistic.
*  <b>`name`</b>: A name for this distribution (optional).


- - -

#### `tf.contrib.distributions.Distribution.allow_nan_stats` {#Distribution.allow_nan_stats}

Python boolean describing behavior when a stat is undefined.

Stats return +/- infinity when it makes sense.  E.g., the variance
of a Cauchy distribution is infinity.  However, sometimes the
statistic is undefined, e.g., if a distribution's pdf does not achieve a
maximum within the support of the distribution, the mode is undefined.
If the mean is undefined, then by definition the variance is undefined.
E.g. the mean for Student's T for df = 1 is undefined (no clear way to say
it is either + or - infinity), so the variance = E[(X - mean)^2] is also
undefined.

##### Returns:


*  <b>`allow_nan_stats`</b>: Python boolean.


- - -

#### `tf.contrib.distributions.Distribution.batch_shape(name='batch_shape')` {#Distribution.batch_shape}

Shape of a single sample from a single event index as a 1-D `Tensor`.

The product of the dimensions of the `batch_shape` is the number of
independent distributions of this kind the instance represents.

##### Args:


*  <b>`name`</b>: name to give to the op

##### Returns:


*  <b>`batch_shape`</b>: `Tensor`.


- - -

#### `tf.contrib.distributions.Distribution.cdf(value, name='cdf')` {#Distribution.cdf}

Cumulative distribution function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`cdf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Distribution.dtype` {#Distribution.dtype}

The `DType` of `Tensor`s handled by this `Distribution`.


- - -

#### `tf.contrib.distributions.Distribution.entropy(name='entropy')` {#Distribution.entropy}

Shanon entropy in nats.


- - -

#### `tf.contrib.distributions.Distribution.event_shape(name='event_shape')` {#Distribution.event_shape}

Shape of a single sample from a single batch as a 1-D int32 `Tensor`.

##### Args:


*  <b>`name`</b>: name to give to the op

##### Returns:


*  <b>`event_shape`</b>: `Tensor`.


- - -

#### `tf.contrib.distributions.Distribution.from_params(cls, make_safe=True, **kwargs)` {#Distribution.from_params}

Given (unconstrained) parameters, return an instantiated distribution.

Subclasses should implement a static method `_safe_transforms` that returns
a dict of parameter transforms, which will be used if `make_safe = True`.

Example usage:

```
# Let's say we want a sample of size (batch_size, 10)
shapes = MultiVariateNormalDiag.param_shapes([batch_size, 10])

# shapes has a Tensor shape for mu and sigma
# shapes == {
#   "mu": tf.constant([batch_size, 10]),
#   "sigma": tf.constant([batch_size, 10]),
# }

# Here we parameterize mu and sigma with the output of a linear
# layer. Note that sigma is unconstrained.
params = {}
for name, shape in shapes.items():
  params[name] = linear(x, shape[1])

# Note that you can forward other kwargs to the `Distribution`, like
# `allow_nan_stats` or `name`.
mvn = MultiVariateNormalDiag.from_params(**params, allow_nan_stats=True)
```

Distribution parameters may have constraints (e.g. `sigma` must be positive
for a `Normal` distribution) and the `from_params` method will apply default
parameter transforms. If a user wants to use their own transform, they can
apply it externally and set `make_safe=False`.

##### Args:


*  <b>`make_safe`</b>: Whether the `params` should be constrained. If True,
    `from_params` will apply default parameter transforms. If False, no
    parameter transforms will be applied.
*  <b>`**kwargs`</b>: dict of parameters for the distribution.

##### Returns:

  A distribution parameterized by possibly transformed parameters in
  `kwargs`.

##### Raises:


*  <b>`TypeError`</b>: if `make_safe` is `True` but `_safe_transforms` is not
    implemented directly for `cls`.


- - -

#### `tf.contrib.distributions.Distribution.get_batch_shape()` {#Distribution.get_batch_shape}

Shape of a single sample from a single event index as a `TensorShape`.

Same meaning as `batch_shape`. May be only partially defined.

##### Returns:


*  <b>`batch_shape`</b>: `TensorShape`, possibly unknown.


- - -

#### `tf.contrib.distributions.Distribution.get_event_shape()` {#Distribution.get_event_shape}

Shape of a single sample from a single batch as a `TensorShape`.

Same meaning as `event_shape`. May be only partially defined.

##### Returns:


*  <b>`event_shape`</b>: `TensorShape`, possibly unknown.


- - -

#### `tf.contrib.distributions.Distribution.is_continuous` {#Distribution.is_continuous}




- - -

#### `tf.contrib.distributions.Distribution.is_reparameterized` {#Distribution.is_reparameterized}




- - -

#### `tf.contrib.distributions.Distribution.log_cdf(value, name='log_cdf')` {#Distribution.log_cdf}

Log cumulative distribution function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`logcdf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Distribution.log_pdf(value, name='log_pdf')` {#Distribution.log_pdf}

Log probability density function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if not `is_continuous`.


- - -

#### `tf.contrib.distributions.Distribution.log_pmf(value, name='log_pmf')` {#Distribution.log_pmf}

Log probability mass function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_pmf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if `is_continuous`.


- - -

#### `tf.contrib.distributions.Distribution.log_prob(value, name='log_prob')` {#Distribution.log_prob}

Log probability density/mass function (depending on `is_continuous`).

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Distribution.mean(name='mean')` {#Distribution.mean}

Mean.


- - -

#### `tf.contrib.distributions.Distribution.mode(name='mode')` {#Distribution.mode}

Mode.


- - -

#### `tf.contrib.distributions.Distribution.name` {#Distribution.name}

Name prepended to all ops created by this `Distribution`.


- - -

#### `tf.contrib.distributions.Distribution.param_shapes(cls, sample_shape, name='DistributionParamShapes')` {#Distribution.param_shapes}

Shapes of parameters given the desired shape of a call to `sample()`.

Subclasses should override static method `_param_shapes`.

##### Args:


*  <b>`sample_shape`</b>: `Tensor` or python list/tuple. Desired shape of a call to
    `sample()`.
*  <b>`name`</b>: name to prepend ops with.

##### Returns:

  `dict` of parameter name to `Tensor` shapes.


- - -

#### `tf.contrib.distributions.Distribution.param_static_shapes(cls, sample_shape)` {#Distribution.param_static_shapes}

param_shapes with static (i.e. TensorShape) shapes.

##### Args:


*  <b>`sample_shape`</b>: `TensorShape` or python list/tuple. Desired shape of a call
    to `sample()`.

##### Returns:

  `dict` of parameter name to `TensorShape`.

##### Raises:


*  <b>`ValueError`</b>: if `sample_shape` is a `TensorShape` and is not fully defined.


- - -

#### `tf.contrib.distributions.Distribution.parameters` {#Distribution.parameters}

Dictionary of parameters used by this `Distribution`.


- - -

#### `tf.contrib.distributions.Distribution.pdf(value, name='pdf')` {#Distribution.pdf}

Probability density function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if not `is_continuous`.


- - -

#### `tf.contrib.distributions.Distribution.pmf(value, name='pmf')` {#Distribution.pmf}

Probability mass function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`pmf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if `is_continuous`.


- - -

#### `tf.contrib.distributions.Distribution.prob(value, name='prob')` {#Distribution.prob}

Probability density/mass function (depending on `is_continuous`).

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Distribution.sample(sample_shape=(), seed=None, name='sample')` {#Distribution.sample}

Generate samples of the specified shape.

Note that a call to `sample()` without arguments will generate a single
sample.

##### Args:


*  <b>`sample_shape`</b>: 0D or 1D `int32` `Tensor`. Shape of the generated samples.
*  <b>`seed`</b>: Python integer seed for RNG
*  <b>`name`</b>: name to give to the op.

##### Returns:


*  <b>`samples`</b>: a `Tensor` with prepended dimensions `sample_shape`.


- - -

#### `tf.contrib.distributions.Distribution.sample_n(n, seed=None, name='sample_n')` {#Distribution.sample_n}

Generate `n` samples.

##### Args:


*  <b>`n`</b>: `Scalar` `Tensor` of type `int32` or `int64`, the number of
    observations to sample.
*  <b>`seed`</b>: Python integer seed for RNG
*  <b>`name`</b>: name to give to the op.

##### Returns:


*  <b>`samples`</b>: a `Tensor` with a prepended dimension (n,).

##### Raises:


*  <b>`TypeError`</b>: if `n` is not an integer type.


- - -

#### `tf.contrib.distributions.Distribution.std(name='std')` {#Distribution.std}

Standard deviation.


- - -

#### `tf.contrib.distributions.Distribution.validate_args` {#Distribution.validate_args}

Python boolean indicated possibly expensive checks are enabled.


- - -

#### `tf.contrib.distributions.Distribution.variance(name='variance')` {#Distribution.variance}

Variance.




### Univariate (scalar) distributions

- - -

### `class tf.contrib.distributions.Binomial` {#Binomial}

Binomial distribution.

This distribution is parameterized by a vector `p` of probabilities and `n`,
the total counts.

#### Mathematical details

The Binomial is a distribution over the number of successes in `n` independent
trials, with each trial having the same probability of success `p`.
The probability mass function (pmf):

```pmf(k) = n! / (k! * (n - k)!) * (p)^k * (1 - p)^(n - k)```

#### Examples

Create a single distribution, corresponding to 5 coin flips.

```python
dist = Binomial(n=5., p=.5)
```

Create a single distribution (using logits), corresponding to 5 coin flips.

```python
dist = Binomial(n=5., logits=0.)
```

Creates 3 distributions with the third distribution most likely to have
successes.

```python
p = [.2, .3, .8]
# n will be broadcast to [4., 4., 4.], to match p.
dist = Binomial(n=4., p=p)
```

The distribution functions can be evaluated on counts.

```python
# counts same shape as p.
counts = [1., 2, 3]
dist.prob(counts)  # Shape [3]

# p will be broadcast to [[.2, .3, .8], [.2, .3, .8]] to match counts.
counts = [[1., 2, 1], [2, 2, 4]]
dist.prob(counts)  # Shape [2, 3]

# p will be broadcast to shape [5, 7, 3] to match counts.
counts = [[...]]  # Shape [5, 7, 3]
dist.prob(counts)  # Shape [5, 7, 3]
```
- - -

#### `tf.contrib.distributions.Binomial.__init__(n, logits=None, p=None, validate_args=True, allow_nan_stats=False, name='Binomial')` {#Binomial.__init__}

Initialize a batch of Binomial distributions.

##### Args:


*  <b>`n`</b>: Non-negative floating point tensor with shape broadcastable to
    `[N1,..., Nm]` with `m >= 0` and the same dtype as `p` or `logits`.
    Defines this as a batch of `N1 x ... x Nm` different Binomial
    distributions. Its components should be equal to integer values.
*  <b>`logits`</b>: Floating point tensor representing the log-odds of a
    positive event with shape broadcastable to `[N1,..., Nm]` `m >= 0`, and
    the same dtype as `n`. Each entry represents logits for the probability
    of success for independent Binomial distributions.
*  <b>`p`</b>: Positive floating point tensor with shape broadcastable to
    `[N1,..., Nm]` `m >= 0`, `p in [0, 1]`. Each entry represents the
    probability of success for independent Binomial distributions.
*  <b>`validate_args`</b>: Whether to assert valid values for parameters `n` and `p`,
    and `x` in `prob` and `log_prob`.  If `False`, correct behavior is not
    guaranteed.
*  <b>`allow_nan_stats`</b>: Boolean, default `False`.  If `False`, raise an
    exception if a statistic (e.g. mean/mode/etc...) is undefined for any
    batch member.  If `True`, batch members with valid parameters leading to
    undefined statistics will return NaN for this statistic.
*  <b>`name`</b>: The name to prefix Ops created by this distribution class.


*  <b>`Examples`</b>: 

```python
# Define 1-batch of a binomial distribution.
dist = Binomial(n=2., p=.9)

# Define a 2-batch.
dist = Binomial(n=[4., 5], p=[.1, .3])
```


- - -

#### `tf.contrib.distributions.Binomial.allow_nan_stats` {#Binomial.allow_nan_stats}

Python boolean describing behavior when a stat is undefined.

Stats return +/- infinity when it makes sense.  E.g., the variance
of a Cauchy distribution is infinity.  However, sometimes the
statistic is undefined, e.g., if a distribution's pdf does not achieve a
maximum within the support of the distribution, the mode is undefined.
If the mean is undefined, then by definition the variance is undefined.
E.g. the mean for Student's T for df = 1 is undefined (no clear way to say
it is either + or - infinity), so the variance = E[(X - mean)^2] is also
undefined.

##### Returns:


*  <b>`allow_nan_stats`</b>: Python boolean.


- - -

#### `tf.contrib.distributions.Binomial.batch_shape(name='batch_shape')` {#Binomial.batch_shape}

Shape of a single sample from a single event index as a 1-D `Tensor`.

The product of the dimensions of the `batch_shape` is the number of
independent distributions of this kind the instance represents.

##### Args:


*  <b>`name`</b>: name to give to the op

##### Returns:


*  <b>`batch_shape`</b>: `Tensor`.


- - -

#### `tf.contrib.distributions.Binomial.cdf(value, name='cdf')` {#Binomial.cdf}

Cumulative distribution function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`cdf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Binomial.dtype` {#Binomial.dtype}

The `DType` of `Tensor`s handled by this `Distribution`.


- - -

#### `tf.contrib.distributions.Binomial.entropy(name='entropy')` {#Binomial.entropy}

Shanon entropy in nats.


- - -

#### `tf.contrib.distributions.Binomial.event_shape(name='event_shape')` {#Binomial.event_shape}

Shape of a single sample from a single batch as a 1-D int32 `Tensor`.

##### Args:


*  <b>`name`</b>: name to give to the op

##### Returns:


*  <b>`event_shape`</b>: `Tensor`.


- - -

#### `tf.contrib.distributions.Binomial.from_params(cls, make_safe=True, **kwargs)` {#Binomial.from_params}

Given (unconstrained) parameters, return an instantiated distribution.

Subclasses should implement a static method `_safe_transforms` that returns
a dict of parameter transforms, which will be used if `make_safe = True`.

Example usage:

```
# Let's say we want a sample of size (batch_size, 10)
shapes = MultiVariateNormalDiag.param_shapes([batch_size, 10])

# shapes has a Tensor shape for mu and sigma
# shapes == {
#   "mu": tf.constant([batch_size, 10]),
#   "sigma": tf.constant([batch_size, 10]),
# }

# Here we parameterize mu and sigma with the output of a linear
# layer. Note that sigma is unconstrained.
params = {}
for name, shape in shapes.items():
  params[name] = linear(x, shape[1])

# Note that you can forward other kwargs to the `Distribution`, like
# `allow_nan_stats` or `name`.
mvn = MultiVariateNormalDiag.from_params(**params, allow_nan_stats=True)
```

Distribution parameters may have constraints (e.g. `sigma` must be positive
for a `Normal` distribution) and the `from_params` method will apply default
parameter transforms. If a user wants to use their own transform, they can
apply it externally and set `make_safe=False`.

##### Args:


*  <b>`make_safe`</b>: Whether the `params` should be constrained. If True,
    `from_params` will apply default parameter transforms. If False, no
    parameter transforms will be applied.
*  <b>`**kwargs`</b>: dict of parameters for the distribution.

##### Returns:

  A distribution parameterized by possibly transformed parameters in
  `kwargs`.

##### Raises:


*  <b>`TypeError`</b>: if `make_safe` is `True` but `_safe_transforms` is not
    implemented directly for `cls`.


- - -

#### `tf.contrib.distributions.Binomial.get_batch_shape()` {#Binomial.get_batch_shape}

Shape of a single sample from a single event index as a `TensorShape`.

Same meaning as `batch_shape`. May be only partially defined.

##### Returns:


*  <b>`batch_shape`</b>: `TensorShape`, possibly unknown.


- - -

#### `tf.contrib.distributions.Binomial.get_event_shape()` {#Binomial.get_event_shape}

Shape of a single sample from a single batch as a `TensorShape`.

Same meaning as `event_shape`. May be only partially defined.

##### Returns:


*  <b>`event_shape`</b>: `TensorShape`, possibly unknown.


- - -

#### `tf.contrib.distributions.Binomial.is_continuous` {#Binomial.is_continuous}




- - -

#### `tf.contrib.distributions.Binomial.is_reparameterized` {#Binomial.is_reparameterized}




- - -

#### `tf.contrib.distributions.Binomial.log_cdf(value, name='log_cdf')` {#Binomial.log_cdf}

Log cumulative distribution function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`logcdf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Binomial.log_pdf(value, name='log_pdf')` {#Binomial.log_pdf}

Log probability density function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if not `is_continuous`.


- - -

#### `tf.contrib.distributions.Binomial.log_pmf(value, name='log_pmf')` {#Binomial.log_pmf}

Log probability mass function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_pmf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if `is_continuous`.


- - -

#### `tf.contrib.distributions.Binomial.log_prob(value, name='log_prob')` {#Binomial.log_prob}

Log probability density/mass function (depending on `is_continuous`).

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Binomial.logits` {#Binomial.logits}

Log-odds.


- - -

#### `tf.contrib.distributions.Binomial.mean(name='mean')` {#Binomial.mean}

Mean.


- - -

#### `tf.contrib.distributions.Binomial.mode(name='mode')` {#Binomial.mode}

Mode.


- - -

#### `tf.contrib.distributions.Binomial.n` {#Binomial.n}

Number of trials.


- - -

#### `tf.contrib.distributions.Binomial.name` {#Binomial.name}

Name prepended to all ops created by this `Distribution`.


- - -

#### `tf.contrib.distributions.Binomial.p` {#Binomial.p}

Probability of success.


- - -

#### `tf.contrib.distributions.Binomial.param_shapes(cls, sample_shape, name='DistributionParamShapes')` {#Binomial.param_shapes}

Shapes of parameters given the desired shape of a call to `sample()`.

Subclasses should override static method `_param_shapes`.

##### Args:


*  <b>`sample_shape`</b>: `Tensor` or python list/tuple. Desired shape of a call to
    `sample()`.
*  <b>`name`</b>: name to prepend ops with.

##### Returns:

  `dict` of parameter name to `Tensor` shapes.


- - -

#### `tf.contrib.distributions.Binomial.param_static_shapes(cls, sample_shape)` {#Binomial.param_static_shapes}

param_shapes with static (i.e. TensorShape) shapes.

##### Args:


*  <b>`sample_shape`</b>: `TensorShape` or python list/tuple. Desired shape of a call
    to `sample()`.

##### Returns:

  `dict` of parameter name to `TensorShape`.

##### Raises:


*  <b>`ValueError`</b>: if `sample_shape` is a `TensorShape` and is not fully defined.


- - -

#### `tf.contrib.distributions.Binomial.parameters` {#Binomial.parameters}

Dictionary of parameters used by this `Distribution`.


- - -

#### `tf.contrib.distributions.Binomial.pdf(value, name='pdf')` {#Binomial.pdf}

Probability density function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if not `is_continuous`.


- - -

#### `tf.contrib.distributions.Binomial.pmf(value, name='pmf')` {#Binomial.pmf}

Probability mass function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`pmf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if `is_continuous`.


- - -

#### `tf.contrib.distributions.Binomial.prob(value, name='prob')` {#Binomial.prob}

Probability density/mass function (depending on `is_continuous`).

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Binomial.sample(sample_shape=(), seed=None, name='sample')` {#Binomial.sample}

Generate samples of the specified shape.

Note that a call to `sample()` without arguments will generate a single
sample.

##### Args:


*  <b>`sample_shape`</b>: 0D or 1D `int32` `Tensor`. Shape of the generated samples.
*  <b>`seed`</b>: Python integer seed for RNG
*  <b>`name`</b>: name to give to the op.

##### Returns:


*  <b>`samples`</b>: a `Tensor` with prepended dimensions `sample_shape`.


- - -

#### `tf.contrib.distributions.Binomial.sample_n(n, seed=None, name='sample_n')` {#Binomial.sample_n}

Generate `n` samples.

##### Args:


*  <b>`n`</b>: `Scalar` `Tensor` of type `int32` or `int64`, the number of
    observations to sample.
*  <b>`seed`</b>: Python integer seed for RNG
*  <b>`name`</b>: name to give to the op.

##### Returns:


*  <b>`samples`</b>: a `Tensor` with a prepended dimension (n,).

##### Raises:


*  <b>`TypeError`</b>: if `n` is not an integer type.


- - -

#### `tf.contrib.distributions.Binomial.std(name='std')` {#Binomial.std}

Standard deviation.


- - -

#### `tf.contrib.distributions.Binomial.validate_args` {#Binomial.validate_args}

Python boolean indicated possibly expensive checks are enabled.


- - -

#### `tf.contrib.distributions.Binomial.variance(name='variance')` {#Binomial.variance}

Variance.



- - -

### `class tf.contrib.distributions.Bernoulli` {#Bernoulli}

Bernoulli distribution.

The Bernoulli distribution is parameterized by p, the probability of a
positive event.
- - -

#### `tf.contrib.distributions.Bernoulli.__init__(logits=None, p=None, dtype=tf.int32, validate_args=True, allow_nan_stats=False, name='Bernoulli')` {#Bernoulli.__init__}

Construct Bernoulli distributions.

##### Args:


*  <b>`logits`</b>: An N-D `Tensor` representing the log-odds
    of a positive event. Each entry in the `Tensor` parametrizes
    an independent Bernoulli distribution where the probability of an event
    is sigmoid(logits).
*  <b>`p`</b>: An N-D `Tensor` representing the probability of a positive
      event. Each entry in the `Tensor` parameterizes an independent
      Bernoulli distribution.
*  <b>`dtype`</b>: dtype for samples.
*  <b>`validate_args`</b>: Whether to assert that `0 <= p <= 1`. If not validate_args,
   `log_pmf` may return nans.
*  <b>`allow_nan_stats`</b>: Boolean, default `False`.  If `False`, raise an
    exception if a statistic (e.g. mean/mode/etc...) is undefined for any
    batch member.  If `True`, batch members with valid parameters leading to
    undefined statistics will return NaN for this statistic.
*  <b>`name`</b>: A name for this distribution.

##### Raises:


*  <b>`ValueError`</b>: If p and logits are passed, or if neither are passed.


- - -

#### `tf.contrib.distributions.Bernoulli.allow_nan_stats` {#Bernoulli.allow_nan_stats}

Python boolean describing behavior when a stat is undefined.

Stats return +/- infinity when it makes sense.  E.g., the variance
of a Cauchy distribution is infinity.  However, sometimes the
statistic is undefined, e.g., if a distribution's pdf does not achieve a
maximum within the support of the distribution, the mode is undefined.
If the mean is undefined, then by definition the variance is undefined.
E.g. the mean for Student's T for df = 1 is undefined (no clear way to say
it is either + or - infinity), so the variance = E[(X - mean)^2] is also
undefined.

##### Returns:


*  <b>`allow_nan_stats`</b>: Python boolean.


- - -

#### `tf.contrib.distributions.Bernoulli.batch_shape(name='batch_shape')` {#Bernoulli.batch_shape}

Shape of a single sample from a single event index as a 1-D `Tensor`.

The product of the dimensions of the `batch_shape` is the number of
independent distributions of this kind the instance represents.

##### Args:


*  <b>`name`</b>: name to give to the op

##### Returns:


*  <b>`batch_shape`</b>: `Tensor`.


- - -

#### `tf.contrib.distributions.Bernoulli.cdf(value, name='cdf')` {#Bernoulli.cdf}

Cumulative distribution function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`cdf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Bernoulli.dtype` {#Bernoulli.dtype}

The `DType` of `Tensor`s handled by this `Distribution`.


- - -

#### `tf.contrib.distributions.Bernoulli.entropy(name='entropy')` {#Bernoulli.entropy}

Shanon entropy in nats.


- - -

#### `tf.contrib.distributions.Bernoulli.event_shape(name='event_shape')` {#Bernoulli.event_shape}

Shape of a single sample from a single batch as a 1-D int32 `Tensor`.

##### Args:


*  <b>`name`</b>: name to give to the op

##### Returns:


*  <b>`event_shape`</b>: `Tensor`.


- - -

#### `tf.contrib.distributions.Bernoulli.from_params(cls, make_safe=True, **kwargs)` {#Bernoulli.from_params}

Given (unconstrained) parameters, return an instantiated distribution.

Subclasses should implement a static method `_safe_transforms` that returns
a dict of parameter transforms, which will be used if `make_safe = True`.

Example usage:

```
# Let's say we want a sample of size (batch_size, 10)
shapes = MultiVariateNormalDiag.param_shapes([batch_size, 10])

# shapes has a Tensor shape for mu and sigma
# shapes == {
#   "mu": tf.constant([batch_size, 10]),
#   "sigma": tf.constant([batch_size, 10]),
# }

# Here we parameterize mu and sigma with the output of a linear
# layer. Note that sigma is unconstrained.
params = {}
for name, shape in shapes.items():
  params[name] = linear(x, shape[1])

# Note that you can forward other kwargs to the `Distribution`, like
# `allow_nan_stats` or `name`.
mvn = MultiVariateNormalDiag.from_params(**params, allow_nan_stats=True)
```

Distribution parameters may have constraints (e.g. `sigma` must be positive
for a `Normal` distribution) and the `from_params` method will apply default
parameter transforms. If a user wants to use their own transform, they can
apply it externally and set `make_safe=False`.

##### Args:


*  <b>`make_safe`</b>: Whether the `params` should be constrained. If True,
    `from_params` will apply default parameter transforms. If False, no
    parameter transforms will be applied.
*  <b>`**kwargs`</b>: dict of parameters for the distribution.

##### Returns:

  A distribution parameterized by possibly transformed parameters in
  `kwargs`.

##### Raises:


*  <b>`TypeError`</b>: if `make_safe` is `True` but `_safe_transforms` is not
    implemented directly for `cls`.


- - -

#### `tf.contrib.distributions.Bernoulli.get_batch_shape()` {#Bernoulli.get_batch_shape}

Shape of a single sample from a single event index as a `TensorShape`.

Same meaning as `batch_shape`. May be only partially defined.

##### Returns:


*  <b>`batch_shape`</b>: `TensorShape`, possibly unknown.


- - -

#### `tf.contrib.distributions.Bernoulli.get_event_shape()` {#Bernoulli.get_event_shape}

Shape of a single sample from a single batch as a `TensorShape`.

Same meaning as `event_shape`. May be only partially defined.

##### Returns:


*  <b>`event_shape`</b>: `TensorShape`, possibly unknown.


- - -

#### `tf.contrib.distributions.Bernoulli.is_continuous` {#Bernoulli.is_continuous}




- - -

#### `tf.contrib.distributions.Bernoulli.is_reparameterized` {#Bernoulli.is_reparameterized}




- - -

#### `tf.contrib.distributions.Bernoulli.log_cdf(value, name='log_cdf')` {#Bernoulli.log_cdf}

Log cumulative distribution function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`logcdf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Bernoulli.log_pdf(value, name='log_pdf')` {#Bernoulli.log_pdf}

Log probability density function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if not `is_continuous`.


- - -

#### `tf.contrib.distributions.Bernoulli.log_pmf(value, name='log_pmf')` {#Bernoulli.log_pmf}

Log probability mass function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_pmf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if `is_continuous`.


- - -

#### `tf.contrib.distributions.Bernoulli.log_prob(value, name='log_prob')` {#Bernoulli.log_prob}

Log probability density/mass function (depending on `is_continuous`).

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Bernoulli.logits` {#Bernoulli.logits}




- - -

#### `tf.contrib.distributions.Bernoulli.mean(name='mean')` {#Bernoulli.mean}

Mean.


- - -

#### `tf.contrib.distributions.Bernoulli.mode(name='mode')` {#Bernoulli.mode}

Mode.


- - -

#### `tf.contrib.distributions.Bernoulli.name` {#Bernoulli.name}

Name prepended to all ops created by this `Distribution`.


- - -

#### `tf.contrib.distributions.Bernoulli.p` {#Bernoulli.p}




- - -

#### `tf.contrib.distributions.Bernoulli.param_shapes(cls, sample_shape, name='DistributionParamShapes')` {#Bernoulli.param_shapes}

Shapes of parameters given the desired shape of a call to `sample()`.

Subclasses should override static method `_param_shapes`.

##### Args:


*  <b>`sample_shape`</b>: `Tensor` or python list/tuple. Desired shape of a call to
    `sample()`.
*  <b>`name`</b>: name to prepend ops with.

##### Returns:

  `dict` of parameter name to `Tensor` shapes.


- - -

#### `tf.contrib.distributions.Bernoulli.param_static_shapes(cls, sample_shape)` {#Bernoulli.param_static_shapes}

param_shapes with static (i.e. TensorShape) shapes.

##### Args:


*  <b>`sample_shape`</b>: `TensorShape` or python list/tuple. Desired shape of a call
    to `sample()`.

##### Returns:

  `dict` of parameter name to `TensorShape`.

##### Raises:


*  <b>`ValueError`</b>: if `sample_shape` is a `TensorShape` and is not fully defined.


- - -

#### `tf.contrib.distributions.Bernoulli.parameters` {#Bernoulli.parameters}

Dictionary of parameters used by this `Distribution`.


- - -

#### `tf.contrib.distributions.Bernoulli.pdf(value, name='pdf')` {#Bernoulli.pdf}

Probability density function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if not `is_continuous`.


- - -

#### `tf.contrib.distributions.Bernoulli.pmf(value, name='pmf')` {#Bernoulli.pmf}

Probability mass function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`pmf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if `is_continuous`.


- - -

#### `tf.contrib.distributions.Bernoulli.prob(value, name='prob')` {#Bernoulli.prob}

Probability density/mass function (depending on `is_continuous`).

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Bernoulli.q` {#Bernoulli.q}

1-p.


- - -

#### `tf.contrib.distributions.Bernoulli.sample(sample_shape=(), seed=None, name='sample')` {#Bernoulli.sample}

Generate samples of the specified shape.

Note that a call to `sample()` without arguments will generate a single
sample.

##### Args:


*  <b>`sample_shape`</b>: 0D or 1D `int32` `Tensor`. Shape of the generated samples.
*  <b>`seed`</b>: Python integer seed for RNG
*  <b>`name`</b>: name to give to the op.

##### Returns:


*  <b>`samples`</b>: a `Tensor` with prepended dimensions `sample_shape`.


- - -

#### `tf.contrib.distributions.Bernoulli.sample_n(n, seed=None, name='sample_n')` {#Bernoulli.sample_n}

Generate `n` samples.

##### Args:


*  <b>`n`</b>: `Scalar` `Tensor` of type `int32` or `int64`, the number of
    observations to sample.
*  <b>`seed`</b>: Python integer seed for RNG
*  <b>`name`</b>: name to give to the op.

##### Returns:


*  <b>`samples`</b>: a `Tensor` with a prepended dimension (n,).

##### Raises:


*  <b>`TypeError`</b>: if `n` is not an integer type.


- - -

#### `tf.contrib.distributions.Bernoulli.std(name='std')` {#Bernoulli.std}

Standard deviation.


- - -

#### `tf.contrib.distributions.Bernoulli.validate_args` {#Bernoulli.validate_args}

Python boolean indicated possibly expensive checks are enabled.


- - -

#### `tf.contrib.distributions.Bernoulli.variance(name='variance')` {#Bernoulli.variance}

Variance.



- - -

### `class tf.contrib.distributions.Beta` {#Beta}

Beta distribution.

This distribution is parameterized by `a` and `b` which are shape
parameters.

#### Mathematical details

The Beta is a distribution over the interval (0, 1).
The distribution has hyperparameters `a` and `b` and
probability mass function (pdf):

```pdf(x) = 1 / Beta(a, b) * x^(a - 1) * (1 - x)^(b - 1)```

where `Beta(a, b) = Gamma(a) * Gamma(b) / Gamma(a + b)`
is the beta function.


This class provides methods to create indexed batches of Beta
distributions. One entry of the broacasted
shape represents of `a` and `b` represents one single Beta distribution.
When calling distribution functions (e.g. `dist.pdf(x)`), `a`, `b`
and `x` are broadcast to the same shape (if possible).
Every entry in a/b/x corresponds to a single Beta distribution.

#### Examples

Creates 3 distributions.
The distribution functions can be evaluated on x.

```python
a = [1, 2, 3]
b = [1, 2, 3]
dist = Beta(a, b)
```

```python
# x same shape as a.
x = [.2, .3, .7]
dist.pdf(x)  # Shape [3]

# a/b will be broadcast to [[1, 2, 3], [1, 2, 3]] to match x.
x = [[.1, .4, .5], [.2, .3, .5]]
dist.pdf(x)  # Shape [2, 3]

# a/b will be broadcast to shape [5, 7, 3] to match x.
x = [[...]]  # Shape [5, 7, 3]
dist.pdf(x)  # Shape [5, 7, 3]
```

Creates a 2-batch of 3-class distributions.

```python
a = [[1, 2, 3], [4, 5, 6]]  # Shape [2, 3]
b = 5  # Shape []
dist = Beta(a, b)

# x will be broadcast to [[.2, .3, .9], [.2, .3, .9]] to match a/b.
x = [.2, .3, .9]
dist.pdf(x)  # Shape [2]
```
- - -

#### `tf.contrib.distributions.Beta.__init__(a, b, validate_args=True, allow_nan_stats=False, name='Beta')` {#Beta.__init__}

Initialize a batch of Beta distributions.

##### Args:


*  <b>`a`</b>: Positive floating point tensor with shape broadcastable to
    `[N1,..., Nm]` `m >= 0`.  Defines this as a batch of `N1 x ... x Nm`
     different Beta distributions. This also defines the
     dtype of the distribution.
*  <b>`b`</b>: Positive floating point tensor with shape broadcastable to
    `[N1,..., Nm]` `m >= 0`.  Defines this as a batch of `N1 x ... x Nm`
     different Beta distributions.
*  <b>`validate_args`</b>: Whether to assert valid values for parameters `a` and `b`,
    and `x` in `prob` and `log_prob`.  If `False`, correct behavior is not
    guaranteed.
*  <b>`allow_nan_stats`</b>: Boolean, default `False`.  If `False`, raise an
    exception if a statistic (e.g. mean/mode/etc...) is undefined for any
    batch member.  If `True`, batch members with valid parameters leading to
    undefined statistics will return NaN for this statistic.
*  <b>`name`</b>: The name to prefix Ops created by this distribution class.


*  <b>`Examples`</b>: 

```python
# Define 1-batch.
dist = Beta(1.1, 2.0)

# Define a 2-batch.
dist = Beta([1.0, 2.0], [4.0, 5.0])
```


- - -

#### `tf.contrib.distributions.Beta.a` {#Beta.a}

Shape parameter.


- - -

#### `tf.contrib.distributions.Beta.a_b_sum` {#Beta.a_b_sum}

Sum of parameters.


- - -

#### `tf.contrib.distributions.Beta.allow_nan_stats` {#Beta.allow_nan_stats}

Python boolean describing behavior when a stat is undefined.

Stats return +/- infinity when it makes sense.  E.g., the variance
of a Cauchy distribution is infinity.  However, sometimes the
statistic is undefined, e.g., if a distribution's pdf does not achieve a
maximum within the support of the distribution, the mode is undefined.
If the mean is undefined, then by definition the variance is undefined.
E.g. the mean for Student's T for df = 1 is undefined (no clear way to say
it is either + or - infinity), so the variance = E[(X - mean)^2] is also
undefined.

##### Returns:


*  <b>`allow_nan_stats`</b>: Python boolean.


- - -

#### `tf.contrib.distributions.Beta.b` {#Beta.b}

Shape parameter.


- - -

#### `tf.contrib.distributions.Beta.batch_shape(name='batch_shape')` {#Beta.batch_shape}

Shape of a single sample from a single event index as a 1-D `Tensor`.

The product of the dimensions of the `batch_shape` is the number of
independent distributions of this kind the instance represents.

##### Args:


*  <b>`name`</b>: name to give to the op

##### Returns:


*  <b>`batch_shape`</b>: `Tensor`.


- - -

#### `tf.contrib.distributions.Beta.cdf(value, name='cdf')` {#Beta.cdf}

Cumulative distribution function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`cdf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Beta.dtype` {#Beta.dtype}

The `DType` of `Tensor`s handled by this `Distribution`.


- - -

#### `tf.contrib.distributions.Beta.entropy(name='entropy')` {#Beta.entropy}

Shanon entropy in nats.


- - -

#### `tf.contrib.distributions.Beta.event_shape(name='event_shape')` {#Beta.event_shape}

Shape of a single sample from a single batch as a 1-D int32 `Tensor`.

##### Args:


*  <b>`name`</b>: name to give to the op

##### Returns:


*  <b>`event_shape`</b>: `Tensor`.


- - -

#### `tf.contrib.distributions.Beta.from_params(cls, make_safe=True, **kwargs)` {#Beta.from_params}

Given (unconstrained) parameters, return an instantiated distribution.

Subclasses should implement a static method `_safe_transforms` that returns
a dict of parameter transforms, which will be used if `make_safe = True`.

Example usage:

```
# Let's say we want a sample of size (batch_size, 10)
shapes = MultiVariateNormalDiag.param_shapes([batch_size, 10])

# shapes has a Tensor shape for mu and sigma
# shapes == {
#   "mu": tf.constant([batch_size, 10]),
#   "sigma": tf.constant([batch_size, 10]),
# }

# Here we parameterize mu and sigma with the output of a linear
# layer. Note that sigma is unconstrained.
params = {}
for name, shape in shapes.items():
  params[name] = linear(x, shape[1])

# Note that you can forward other kwargs to the `Distribution`, like
# `allow_nan_stats` or `name`.
mvn = MultiVariateNormalDiag.from_params(**params, allow_nan_stats=True)
```

Distribution parameters may have constraints (e.g. `sigma` must be positive
for a `Normal` distribution) and the `from_params` method will apply default
parameter transforms. If a user wants to use their own transform, they can
apply it externally and set `make_safe=False`.

##### Args:


*  <b>`make_safe`</b>: Whether the `params` should be constrained. If True,
    `from_params` will apply default parameter transforms. If False, no
    parameter transforms will be applied.
*  <b>`**kwargs`</b>: dict of parameters for the distribution.

##### Returns:

  A distribution parameterized by possibly transformed parameters in
  `kwargs`.

##### Raises:


*  <b>`TypeError`</b>: if `make_safe` is `True` but `_safe_transforms` is not
    implemented directly for `cls`.


- - -

#### `tf.contrib.distributions.Beta.get_batch_shape()` {#Beta.get_batch_shape}

Shape of a single sample from a single event index as a `TensorShape`.

Same meaning as `batch_shape`. May be only partially defined.

##### Returns:


*  <b>`batch_shape`</b>: `TensorShape`, possibly unknown.


- - -

#### `tf.contrib.distributions.Beta.get_event_shape()` {#Beta.get_event_shape}

Shape of a single sample from a single batch as a `TensorShape`.

Same meaning as `event_shape`. May be only partially defined.

##### Returns:


*  <b>`event_shape`</b>: `TensorShape`, possibly unknown.


- - -

#### `tf.contrib.distributions.Beta.is_continuous` {#Beta.is_continuous}




- - -

#### `tf.contrib.distributions.Beta.is_reparameterized` {#Beta.is_reparameterized}




- - -

#### `tf.contrib.distributions.Beta.log_cdf(value, name='log_cdf')` {#Beta.log_cdf}

Log cumulative distribution function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`logcdf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Beta.log_pdf(value, name='log_pdf')` {#Beta.log_pdf}

Log probability density function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if not `is_continuous`.


- - -

#### `tf.contrib.distributions.Beta.log_pmf(value, name='log_pmf')` {#Beta.log_pmf}

Log probability mass function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_pmf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if `is_continuous`.


- - -

#### `tf.contrib.distributions.Beta.log_prob(value, name='log_prob')` {#Beta.log_prob}

Log probability density/mass function (depending on `is_continuous`).

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Beta.mean(name='mean')` {#Beta.mean}

Mean.


- - -

#### `tf.contrib.distributions.Beta.mode(name='mode')` {#Beta.mode}

Mode.


- - -

#### `tf.contrib.distributions.Beta.name` {#Beta.name}

Name prepended to all ops created by this `Distribution`.


- - -

#### `tf.contrib.distributions.Beta.param_shapes(cls, sample_shape, name='DistributionParamShapes')` {#Beta.param_shapes}

Shapes of parameters given the desired shape of a call to `sample()`.

Subclasses should override static method `_param_shapes`.

##### Args:


*  <b>`sample_shape`</b>: `Tensor` or python list/tuple. Desired shape of a call to
    `sample()`.
*  <b>`name`</b>: name to prepend ops with.

##### Returns:

  `dict` of parameter name to `Tensor` shapes.


- - -

#### `tf.contrib.distributions.Beta.param_static_shapes(cls, sample_shape)` {#Beta.param_static_shapes}

param_shapes with static (i.e. TensorShape) shapes.

##### Args:


*  <b>`sample_shape`</b>: `TensorShape` or python list/tuple. Desired shape of a call
    to `sample()`.

##### Returns:

  `dict` of parameter name to `TensorShape`.

##### Raises:


*  <b>`ValueError`</b>: if `sample_shape` is a `TensorShape` and is not fully defined.


- - -

#### `tf.contrib.distributions.Beta.parameters` {#Beta.parameters}

Dictionary of parameters used by this `Distribution`.


- - -

#### `tf.contrib.distributions.Beta.pdf(value, name='pdf')` {#Beta.pdf}

Probability density function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if not `is_continuous`.


- - -

#### `tf.contrib.distributions.Beta.pmf(value, name='pmf')` {#Beta.pmf}

Probability mass function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`pmf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if `is_continuous`.


- - -

#### `tf.contrib.distributions.Beta.prob(value, name='prob')` {#Beta.prob}

Probability density/mass function (depending on `is_continuous`).

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Beta.sample(sample_shape=(), seed=None, name='sample')` {#Beta.sample}

Generate samples of the specified shape.

Note that a call to `sample()` without arguments will generate a single
sample.

##### Args:


*  <b>`sample_shape`</b>: 0D or 1D `int32` `Tensor`. Shape of the generated samples.
*  <b>`seed`</b>: Python integer seed for RNG
*  <b>`name`</b>: name to give to the op.

##### Returns:


*  <b>`samples`</b>: a `Tensor` with prepended dimensions `sample_shape`.


- - -

#### `tf.contrib.distributions.Beta.sample_n(n, seed=None, name='sample_n')` {#Beta.sample_n}

Generate `n` samples.

##### Args:


*  <b>`n`</b>: `Scalar` `Tensor` of type `int32` or `int64`, the number of
    observations to sample.
*  <b>`seed`</b>: Python integer seed for RNG
*  <b>`name`</b>: name to give to the op.

##### Returns:


*  <b>`samples`</b>: a `Tensor` with a prepended dimension (n,).

##### Raises:


*  <b>`TypeError`</b>: if `n` is not an integer type.


- - -

#### `tf.contrib.distributions.Beta.std(name='std')` {#Beta.std}

Standard deviation.


- - -

#### `tf.contrib.distributions.Beta.validate_args` {#Beta.validate_args}

Python boolean indicated possibly expensive checks are enabled.


- - -

#### `tf.contrib.distributions.Beta.variance(name='variance')` {#Beta.variance}

Variance.



- - -

### `class tf.contrib.distributions.Categorical` {#Categorical}

Categorical distribution.

The categorical distribution is parameterized by the log-probabilities
of a set of classes.
- - -

#### `tf.contrib.distributions.Categorical.__init__(logits, dtype=tf.int32, validate_args=True, allow_nan_stats=False, name='Categorical')` {#Categorical.__init__}

Initialize Categorical distributions using class log-probabilities.

##### Args:


*  <b>`logits`</b>: An N-D `Tensor`, `N >= 1`, representing the log probabilities
      of a set of Categorical distributions. The first `N - 1` dimensions
      index into a batch of independent distributions and the last dimension
      indexes into the classes.
*  <b>`dtype`</b>: The type of the event samples (default: int32).
*  <b>`validate_args`</b>: Unused in this distribution.
*  <b>`allow_nan_stats`</b>: Boolean, default `False`.  If `False`, raise an
    exception if a statistic (e.g. mean/mode/etc...) is undefined for any
    batch member.  If `True`, batch members with valid parameters leading to
    undefined statistics will return NaN for this statistic.
*  <b>`name`</b>: A name for this distribution (optional).


- - -

#### `tf.contrib.distributions.Categorical.allow_nan_stats` {#Categorical.allow_nan_stats}

Python boolean describing behavior when a stat is undefined.

Stats return +/- infinity when it makes sense.  E.g., the variance
of a Cauchy distribution is infinity.  However, sometimes the
statistic is undefined, e.g., if a distribution's pdf does not achieve a
maximum within the support of the distribution, the mode is undefined.
If the mean is undefined, then by definition the variance is undefined.
E.g. the mean for Student's T for df = 1 is undefined (no clear way to say
it is either + or - infinity), so the variance = E[(X - mean)^2] is also
undefined.

##### Returns:


*  <b>`allow_nan_stats`</b>: Python boolean.


- - -

#### `tf.contrib.distributions.Categorical.batch_shape(name='batch_shape')` {#Categorical.batch_shape}

Shape of a single sample from a single event index as a 1-D `Tensor`.

The product of the dimensions of the `batch_shape` is the number of
independent distributions of this kind the instance represents.

##### Args:


*  <b>`name`</b>: name to give to the op

##### Returns:


*  <b>`batch_shape`</b>: `Tensor`.


- - -

#### `tf.contrib.distributions.Categorical.cdf(value, name='cdf')` {#Categorical.cdf}

Cumulative distribution function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`cdf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Categorical.dtype` {#Categorical.dtype}

The `DType` of `Tensor`s handled by this `Distribution`.


- - -

#### `tf.contrib.distributions.Categorical.entropy(name='entropy')` {#Categorical.entropy}

Shanon entropy in nats.


- - -

#### `tf.contrib.distributions.Categorical.event_shape(name='event_shape')` {#Categorical.event_shape}

Shape of a single sample from a single batch as a 1-D int32 `Tensor`.

##### Args:


*  <b>`name`</b>: name to give to the op

##### Returns:


*  <b>`event_shape`</b>: `Tensor`.


- - -

#### `tf.contrib.distributions.Categorical.from_params(cls, make_safe=True, **kwargs)` {#Categorical.from_params}

Given (unconstrained) parameters, return an instantiated distribution.

Subclasses should implement a static method `_safe_transforms` that returns
a dict of parameter transforms, which will be used if `make_safe = True`.

Example usage:

```
# Let's say we want a sample of size (batch_size, 10)
shapes = MultiVariateNormalDiag.param_shapes([batch_size, 10])

# shapes has a Tensor shape for mu and sigma
# shapes == {
#   "mu": tf.constant([batch_size, 10]),
#   "sigma": tf.constant([batch_size, 10]),
# }

# Here we parameterize mu and sigma with the output of a linear
# layer. Note that sigma is unconstrained.
params = {}
for name, shape in shapes.items():
  params[name] = linear(x, shape[1])

# Note that you can forward other kwargs to the `Distribution`, like
# `allow_nan_stats` or `name`.
mvn = MultiVariateNormalDiag.from_params(**params, allow_nan_stats=True)
```

Distribution parameters may have constraints (e.g. `sigma` must be positive
for a `Normal` distribution) and the `from_params` method will apply default
parameter transforms. If a user wants to use their own transform, they can
apply it externally and set `make_safe=False`.

##### Args:


*  <b>`make_safe`</b>: Whether the `params` should be constrained. If True,
    `from_params` will apply default parameter transforms. If False, no
    parameter transforms will be applied.
*  <b>`**kwargs`</b>: dict of parameters for the distribution.

##### Returns:

  A distribution parameterized by possibly transformed parameters in
  `kwargs`.

##### Raises:


*  <b>`TypeError`</b>: if `make_safe` is `True` but `_safe_transforms` is not
    implemented directly for `cls`.


- - -

#### `tf.contrib.distributions.Categorical.get_batch_shape()` {#Categorical.get_batch_shape}

Shape of a single sample from a single event index as a `TensorShape`.

Same meaning as `batch_shape`. May be only partially defined.

##### Returns:


*  <b>`batch_shape`</b>: `TensorShape`, possibly unknown.


- - -

#### `tf.contrib.distributions.Categorical.get_event_shape()` {#Categorical.get_event_shape}

Shape of a single sample from a single batch as a `TensorShape`.

Same meaning as `event_shape`. May be only partially defined.

##### Returns:


*  <b>`event_shape`</b>: `TensorShape`, possibly unknown.


- - -

#### `tf.contrib.distributions.Categorical.is_continuous` {#Categorical.is_continuous}




- - -

#### `tf.contrib.distributions.Categorical.is_reparameterized` {#Categorical.is_reparameterized}




- - -

#### `tf.contrib.distributions.Categorical.log_cdf(value, name='log_cdf')` {#Categorical.log_cdf}

Log cumulative distribution function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`logcdf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Categorical.log_pdf(value, name='log_pdf')` {#Categorical.log_pdf}

Log probability density function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if not `is_continuous`.


- - -

#### `tf.contrib.distributions.Categorical.log_pmf(value, name='log_pmf')` {#Categorical.log_pmf}

Log probability mass function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_pmf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if `is_continuous`.


- - -

#### `tf.contrib.distributions.Categorical.log_prob(value, name='log_prob')` {#Categorical.log_prob}

Log probability density/mass function (depending on `is_continuous`).

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Categorical.logits` {#Categorical.logits}




- - -

#### `tf.contrib.distributions.Categorical.mean(name='mean')` {#Categorical.mean}

Mean.


- - -

#### `tf.contrib.distributions.Categorical.mode(name='mode')` {#Categorical.mode}

Mode.


- - -

#### `tf.contrib.distributions.Categorical.name` {#Categorical.name}

Name prepended to all ops created by this `Distribution`.


- - -

#### `tf.contrib.distributions.Categorical.num_classes` {#Categorical.num_classes}

Scalar `int32` tensor: the number of classes.


- - -

#### `tf.contrib.distributions.Categorical.param_shapes(cls, sample_shape, name='DistributionParamShapes')` {#Categorical.param_shapes}

Shapes of parameters given the desired shape of a call to `sample()`.

Subclasses should override static method `_param_shapes`.

##### Args:


*  <b>`sample_shape`</b>: `Tensor` or python list/tuple. Desired shape of a call to
    `sample()`.
*  <b>`name`</b>: name to prepend ops with.

##### Returns:

  `dict` of parameter name to `Tensor` shapes.


- - -

#### `tf.contrib.distributions.Categorical.param_static_shapes(cls, sample_shape)` {#Categorical.param_static_shapes}

param_shapes with static (i.e. TensorShape) shapes.

##### Args:


*  <b>`sample_shape`</b>: `TensorShape` or python list/tuple. Desired shape of a call
    to `sample()`.

##### Returns:

  `dict` of parameter name to `TensorShape`.

##### Raises:


*  <b>`ValueError`</b>: if `sample_shape` is a `TensorShape` and is not fully defined.


- - -

#### `tf.contrib.distributions.Categorical.parameters` {#Categorical.parameters}

Dictionary of parameters used by this `Distribution`.


- - -

#### `tf.contrib.distributions.Categorical.pdf(value, name='pdf')` {#Categorical.pdf}

Probability density function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if not `is_continuous`.


- - -

#### `tf.contrib.distributions.Categorical.pmf(value, name='pmf')` {#Categorical.pmf}

Probability mass function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`pmf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if `is_continuous`.


- - -

#### `tf.contrib.distributions.Categorical.prob(value, name='prob')` {#Categorical.prob}

Probability density/mass function (depending on `is_continuous`).

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Categorical.sample(sample_shape=(), seed=None, name='sample')` {#Categorical.sample}

Generate samples of the specified shape.

Note that a call to `sample()` without arguments will generate a single
sample.

##### Args:


*  <b>`sample_shape`</b>: 0D or 1D `int32` `Tensor`. Shape of the generated samples.
*  <b>`seed`</b>: Python integer seed for RNG
*  <b>`name`</b>: name to give to the op.

##### Returns:


*  <b>`samples`</b>: a `Tensor` with prepended dimensions `sample_shape`.


- - -

#### `tf.contrib.distributions.Categorical.sample_n(n, seed=None, name='sample_n')` {#Categorical.sample_n}

Generate `n` samples.

##### Args:


*  <b>`n`</b>: `Scalar` `Tensor` of type `int32` or `int64`, the number of
    observations to sample.
*  <b>`seed`</b>: Python integer seed for RNG
*  <b>`name`</b>: name to give to the op.

##### Returns:


*  <b>`samples`</b>: a `Tensor` with a prepended dimension (n,).

##### Raises:


*  <b>`TypeError`</b>: if `n` is not an integer type.


- - -

#### `tf.contrib.distributions.Categorical.std(name='std')` {#Categorical.std}

Standard deviation.


- - -

#### `tf.contrib.distributions.Categorical.validate_args` {#Categorical.validate_args}

Python boolean indicated possibly expensive checks are enabled.


- - -

#### `tf.contrib.distributions.Categorical.variance(name='variance')` {#Categorical.variance}

Variance.



- - -

### `class tf.contrib.distributions.Chi2` {#Chi2}

The Chi2 distribution with degrees of freedom df.

The PDF of this distribution is:

```pdf(x) = (x^(df/2 - 1)e^(-x/2))/(2^(df/2)Gamma(df/2)), x > 0```

Note that the Chi2 distribution is a special case of the Gamma distribution,
with Chi2(df) = Gamma(df/2, 1/2).
- - -

#### `tf.contrib.distributions.Chi2.__init__(df, validate_args=True, allow_nan_stats=False, name='Chi2')` {#Chi2.__init__}

Construct Chi2 distributions with parameter `df`.

##### Args:


*  <b>`df`</b>: Floating point tensor, the degrees of freedom of the
    distribution(s).  `df` must contain only positive values.
*  <b>`validate_args`</b>: Whether to assert that `df > 0`, and that `x > 0` in the
    methods `prob(x)` and `log_prob(x)`. If `validate_args` is `False`
    and the inputs are invalid, correct behavior is not guaranteed.
*  <b>`allow_nan_stats`</b>: Boolean, default `False`.  If `False`, raise an
    exception if a statistic (e.g. mean/mode/etc...) is undefined for any
    batch member.  If `True`, batch members with valid parameters leading to
    undefined statistics will return NaN for this statistic.
*  <b>`name`</b>: The name to prepend to all ops created by this distribution.


- - -

#### `tf.contrib.distributions.Chi2.allow_nan_stats` {#Chi2.allow_nan_stats}

Python boolean describing behavior when a stat is undefined.

Stats return +/- infinity when it makes sense.  E.g., the variance
of a Cauchy distribution is infinity.  However, sometimes the
statistic is undefined, e.g., if a distribution's pdf does not achieve a
maximum within the support of the distribution, the mode is undefined.
If the mean is undefined, then by definition the variance is undefined.
E.g. the mean for Student's T for df = 1 is undefined (no clear way to say
it is either + or - infinity), so the variance = E[(X - mean)^2] is also
undefined.

##### Returns:


*  <b>`allow_nan_stats`</b>: Python boolean.


- - -

#### `tf.contrib.distributions.Chi2.alpha` {#Chi2.alpha}

Shape parameter.


- - -

#### `tf.contrib.distributions.Chi2.batch_shape(name='batch_shape')` {#Chi2.batch_shape}

Shape of a single sample from a single event index as a 1-D `Tensor`.

The product of the dimensions of the `batch_shape` is the number of
independent distributions of this kind the instance represents.

##### Args:


*  <b>`name`</b>: name to give to the op

##### Returns:


*  <b>`batch_shape`</b>: `Tensor`.


- - -

#### `tf.contrib.distributions.Chi2.beta` {#Chi2.beta}

Inverse scale parameter.


- - -

#### `tf.contrib.distributions.Chi2.cdf(value, name='cdf')` {#Chi2.cdf}

Cumulative distribution function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`cdf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Chi2.df` {#Chi2.df}




- - -

#### `tf.contrib.distributions.Chi2.dtype` {#Chi2.dtype}

The `DType` of `Tensor`s handled by this `Distribution`.


- - -

#### `tf.contrib.distributions.Chi2.entropy(name='entropy')` {#Chi2.entropy}

Shanon entropy in nats.


- - -

#### `tf.contrib.distributions.Chi2.event_shape(name='event_shape')` {#Chi2.event_shape}

Shape of a single sample from a single batch as a 1-D int32 `Tensor`.

##### Args:


*  <b>`name`</b>: name to give to the op

##### Returns:


*  <b>`event_shape`</b>: `Tensor`.


- - -

#### `tf.contrib.distributions.Chi2.from_params(cls, make_safe=True, **kwargs)` {#Chi2.from_params}

Given (unconstrained) parameters, return an instantiated distribution.

Subclasses should implement a static method `_safe_transforms` that returns
a dict of parameter transforms, which will be used if `make_safe = True`.

Example usage:

```
# Let's say we want a sample of size (batch_size, 10)
shapes = MultiVariateNormalDiag.param_shapes([batch_size, 10])

# shapes has a Tensor shape for mu and sigma
# shapes == {
#   "mu": tf.constant([batch_size, 10]),
#   "sigma": tf.constant([batch_size, 10]),
# }

# Here we parameterize mu and sigma with the output of a linear
# layer. Note that sigma is unconstrained.
params = {}
for name, shape in shapes.items():
  params[name] = linear(x, shape[1])

# Note that you can forward other kwargs to the `Distribution`, like
# `allow_nan_stats` or `name`.
mvn = MultiVariateNormalDiag.from_params(**params, allow_nan_stats=True)
```

Distribution parameters may have constraints (e.g. `sigma` must be positive
for a `Normal` distribution) and the `from_params` method will apply default
parameter transforms. If a user wants to use their own transform, they can
apply it externally and set `make_safe=False`.

##### Args:


*  <b>`make_safe`</b>: Whether the `params` should be constrained. If True,
    `from_params` will apply default parameter transforms. If False, no
    parameter transforms will be applied.
*  <b>`**kwargs`</b>: dict of parameters for the distribution.

##### Returns:

  A distribution parameterized by possibly transformed parameters in
  `kwargs`.

##### Raises:


*  <b>`TypeError`</b>: if `make_safe` is `True` but `_safe_transforms` is not
    implemented directly for `cls`.


- - -

#### `tf.contrib.distributions.Chi2.get_batch_shape()` {#Chi2.get_batch_shape}

Shape of a single sample from a single event index as a `TensorShape`.

Same meaning as `batch_shape`. May be only partially defined.

##### Returns:


*  <b>`batch_shape`</b>: `TensorShape`, possibly unknown.


- - -

#### `tf.contrib.distributions.Chi2.get_event_shape()` {#Chi2.get_event_shape}

Shape of a single sample from a single batch as a `TensorShape`.

Same meaning as `event_shape`. May be only partially defined.

##### Returns:


*  <b>`event_shape`</b>: `TensorShape`, possibly unknown.


- - -

#### `tf.contrib.distributions.Chi2.is_continuous` {#Chi2.is_continuous}




- - -

#### `tf.contrib.distributions.Chi2.is_reparameterized` {#Chi2.is_reparameterized}




- - -

#### `tf.contrib.distributions.Chi2.log_cdf(value, name='log_cdf')` {#Chi2.log_cdf}

Log cumulative distribution function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`logcdf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Chi2.log_pdf(value, name='log_pdf')` {#Chi2.log_pdf}

Log probability density function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if not `is_continuous`.


- - -

#### `tf.contrib.distributions.Chi2.log_pmf(value, name='log_pmf')` {#Chi2.log_pmf}

Log probability mass function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_pmf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if `is_continuous`.


- - -

#### `tf.contrib.distributions.Chi2.log_prob(value, name='log_prob')` {#Chi2.log_prob}

Log probability density/mass function (depending on `is_continuous`).

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Chi2.mean(name='mean')` {#Chi2.mean}

Mean.


- - -

#### `tf.contrib.distributions.Chi2.mode(name='mode')` {#Chi2.mode}

Mode.


- - -

#### `tf.contrib.distributions.Chi2.name` {#Chi2.name}

Name prepended to all ops created by this `Distribution`.


- - -

#### `tf.contrib.distributions.Chi2.param_shapes(cls, sample_shape, name='DistributionParamShapes')` {#Chi2.param_shapes}

Shapes of parameters given the desired shape of a call to `sample()`.

Subclasses should override static method `_param_shapes`.

##### Args:


*  <b>`sample_shape`</b>: `Tensor` or python list/tuple. Desired shape of a call to
    `sample()`.
*  <b>`name`</b>: name to prepend ops with.

##### Returns:

  `dict` of parameter name to `Tensor` shapes.


- - -

#### `tf.contrib.distributions.Chi2.param_static_shapes(cls, sample_shape)` {#Chi2.param_static_shapes}

param_shapes with static (i.e. TensorShape) shapes.

##### Args:


*  <b>`sample_shape`</b>: `TensorShape` or python list/tuple. Desired shape of a call
    to `sample()`.

##### Returns:

  `dict` of parameter name to `TensorShape`.

##### Raises:


*  <b>`ValueError`</b>: if `sample_shape` is a `TensorShape` and is not fully defined.


- - -

#### `tf.contrib.distributions.Chi2.parameters` {#Chi2.parameters}

Dictionary of parameters used by this `Distribution`.


- - -

#### `tf.contrib.distributions.Chi2.pdf(value, name='pdf')` {#Chi2.pdf}

Probability density function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if not `is_continuous`.


- - -

#### `tf.contrib.distributions.Chi2.pmf(value, name='pmf')` {#Chi2.pmf}

Probability mass function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`pmf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if `is_continuous`.


- - -

#### `tf.contrib.distributions.Chi2.prob(value, name='prob')` {#Chi2.prob}

Probability density/mass function (depending on `is_continuous`).

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Chi2.sample(sample_shape=(), seed=None, name='sample')` {#Chi2.sample}

Generate samples of the specified shape.

Note that a call to `sample()` without arguments will generate a single
sample.

##### Args:


*  <b>`sample_shape`</b>: 0D or 1D `int32` `Tensor`. Shape of the generated samples.
*  <b>`seed`</b>: Python integer seed for RNG
*  <b>`name`</b>: name to give to the op.

##### Returns:


*  <b>`samples`</b>: a `Tensor` with prepended dimensions `sample_shape`.


- - -

#### `tf.contrib.distributions.Chi2.sample_n(n, seed=None, name='sample_n')` {#Chi2.sample_n}

Generate `n` samples.

##### Args:


*  <b>`n`</b>: `Scalar` `Tensor` of type `int32` or `int64`, the number of
    observations to sample.
*  <b>`seed`</b>: Python integer seed for RNG
*  <b>`name`</b>: name to give to the op.

##### Returns:


*  <b>`samples`</b>: a `Tensor` with a prepended dimension (n,).

##### Raises:


*  <b>`TypeError`</b>: if `n` is not an integer type.


- - -

#### `tf.contrib.distributions.Chi2.std(name='std')` {#Chi2.std}

Standard deviation.


- - -

#### `tf.contrib.distributions.Chi2.validate_args` {#Chi2.validate_args}

Python boolean indicated possibly expensive checks are enabled.


- - -

#### `tf.contrib.distributions.Chi2.variance(name='variance')` {#Chi2.variance}

Variance.



- - -

### `class tf.contrib.distributions.Exponential` {#Exponential}

The Exponential distribution with rate parameter lam.

The PDF of this distribution is:

```prob(x) = (lam * e^(-lam * x)), x > 0```

Note that the Exponential distribution is a special case of the Gamma
distribution, with Exponential(lam) = Gamma(1, lam).
- - -

#### `tf.contrib.distributions.Exponential.__init__(lam, validate_args=True, allow_nan_stats=False, name='Exponential')` {#Exponential.__init__}

Construct Exponential distribution with parameter `lam`.

##### Args:


*  <b>`lam`</b>: Floating point tensor, the rate of the distribution(s).
    `lam` must contain only positive values.
*  <b>`validate_args`</b>: Whether to assert that `lam > 0`, and that `x > 0` in the
    methods `prob(x)` and `log_prob(x)`.  If `validate_args` is `False`
    and the inputs are invalid, correct behavior is not guaranteed.
*  <b>`allow_nan_stats`</b>: Boolean, default `False`.  If `False`, raise an
    exception if a statistic (e.g. mean/mode/etc...) is undefined for any
    batch member. If `True`, batch members with valid parameters leading to
    undefined statistics will return NaN for this statistic.
*  <b>`name`</b>: The name to prepend to all ops created by this distribution.


- - -

#### `tf.contrib.distributions.Exponential.allow_nan_stats` {#Exponential.allow_nan_stats}

Python boolean describing behavior when a stat is undefined.

Stats return +/- infinity when it makes sense.  E.g., the variance
of a Cauchy distribution is infinity.  However, sometimes the
statistic is undefined, e.g., if a distribution's pdf does not achieve a
maximum within the support of the distribution, the mode is undefined.
If the mean is undefined, then by definition the variance is undefined.
E.g. the mean for Student's T for df = 1 is undefined (no clear way to say
it is either + or - infinity), so the variance = E[(X - mean)^2] is also
undefined.

##### Returns:


*  <b>`allow_nan_stats`</b>: Python boolean.


- - -

#### `tf.contrib.distributions.Exponential.alpha` {#Exponential.alpha}

Shape parameter.


- - -

#### `tf.contrib.distributions.Exponential.batch_shape(name='batch_shape')` {#Exponential.batch_shape}

Shape of a single sample from a single event index as a 1-D `Tensor`.

The product of the dimensions of the `batch_shape` is the number of
independent distributions of this kind the instance represents.

##### Args:


*  <b>`name`</b>: name to give to the op

##### Returns:


*  <b>`batch_shape`</b>: `Tensor`.


- - -

#### `tf.contrib.distributions.Exponential.beta` {#Exponential.beta}

Inverse scale parameter.


- - -

#### `tf.contrib.distributions.Exponential.cdf(value, name='cdf')` {#Exponential.cdf}

Cumulative distribution function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`cdf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Exponential.dtype` {#Exponential.dtype}

The `DType` of `Tensor`s handled by this `Distribution`.


- - -

#### `tf.contrib.distributions.Exponential.entropy(name='entropy')` {#Exponential.entropy}

Shanon entropy in nats.


- - -

#### `tf.contrib.distributions.Exponential.event_shape(name='event_shape')` {#Exponential.event_shape}

Shape of a single sample from a single batch as a 1-D int32 `Tensor`.

##### Args:


*  <b>`name`</b>: name to give to the op

##### Returns:


*  <b>`event_shape`</b>: `Tensor`.


- - -

#### `tf.contrib.distributions.Exponential.from_params(cls, make_safe=True, **kwargs)` {#Exponential.from_params}

Given (unconstrained) parameters, return an instantiated distribution.

Subclasses should implement a static method `_safe_transforms` that returns
a dict of parameter transforms, which will be used if `make_safe = True`.

Example usage:

```
# Let's say we want a sample of size (batch_size, 10)
shapes = MultiVariateNormalDiag.param_shapes([batch_size, 10])

# shapes has a Tensor shape for mu and sigma
# shapes == {
#   "mu": tf.constant([batch_size, 10]),
#   "sigma": tf.constant([batch_size, 10]),
# }

# Here we parameterize mu and sigma with the output of a linear
# layer. Note that sigma is unconstrained.
params = {}
for name, shape in shapes.items():
  params[name] = linear(x, shape[1])

# Note that you can forward other kwargs to the `Distribution`, like
# `allow_nan_stats` or `name`.
mvn = MultiVariateNormalDiag.from_params(**params, allow_nan_stats=True)
```

Distribution parameters may have constraints (e.g. `sigma` must be positive
for a `Normal` distribution) and the `from_params` method will apply default
parameter transforms. If a user wants to use their own transform, they can
apply it externally and set `make_safe=False`.

##### Args:


*  <b>`make_safe`</b>: Whether the `params` should be constrained. If True,
    `from_params` will apply default parameter transforms. If False, no
    parameter transforms will be applied.
*  <b>`**kwargs`</b>: dict of parameters for the distribution.

##### Returns:

  A distribution parameterized by possibly transformed parameters in
  `kwargs`.

##### Raises:


*  <b>`TypeError`</b>: if `make_safe` is `True` but `_safe_transforms` is not
    implemented directly for `cls`.


- - -

#### `tf.contrib.distributions.Exponential.get_batch_shape()` {#Exponential.get_batch_shape}

Shape of a single sample from a single event index as a `TensorShape`.

Same meaning as `batch_shape`. May be only partially defined.

##### Returns:


*  <b>`batch_shape`</b>: `TensorShape`, possibly unknown.


- - -

#### `tf.contrib.distributions.Exponential.get_event_shape()` {#Exponential.get_event_shape}

Shape of a single sample from a single batch as a `TensorShape`.

Same meaning as `event_shape`. May be only partially defined.

##### Returns:


*  <b>`event_shape`</b>: `TensorShape`, possibly unknown.


- - -

#### `tf.contrib.distributions.Exponential.is_continuous` {#Exponential.is_continuous}




- - -

#### `tf.contrib.distributions.Exponential.is_reparameterized` {#Exponential.is_reparameterized}




- - -

#### `tf.contrib.distributions.Exponential.lam` {#Exponential.lam}




- - -

#### `tf.contrib.distributions.Exponential.log_cdf(value, name='log_cdf')` {#Exponential.log_cdf}

Log cumulative distribution function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`logcdf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Exponential.log_pdf(value, name='log_pdf')` {#Exponential.log_pdf}

Log probability density function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if not `is_continuous`.


- - -

#### `tf.contrib.distributions.Exponential.log_pmf(value, name='log_pmf')` {#Exponential.log_pmf}

Log probability mass function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_pmf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if `is_continuous`.


- - -

#### `tf.contrib.distributions.Exponential.log_prob(value, name='log_prob')` {#Exponential.log_prob}

Log probability density/mass function (depending on `is_continuous`).

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Exponential.mean(name='mean')` {#Exponential.mean}

Mean.


- - -

#### `tf.contrib.distributions.Exponential.mode(name='mode')` {#Exponential.mode}

Mode.


- - -

#### `tf.contrib.distributions.Exponential.name` {#Exponential.name}

Name prepended to all ops created by this `Distribution`.


- - -

#### `tf.contrib.distributions.Exponential.param_shapes(cls, sample_shape, name='DistributionParamShapes')` {#Exponential.param_shapes}

Shapes of parameters given the desired shape of a call to `sample()`.

Subclasses should override static method `_param_shapes`.

##### Args:


*  <b>`sample_shape`</b>: `Tensor` or python list/tuple. Desired shape of a call to
    `sample()`.
*  <b>`name`</b>: name to prepend ops with.

##### Returns:

  `dict` of parameter name to `Tensor` shapes.


- - -

#### `tf.contrib.distributions.Exponential.param_static_shapes(cls, sample_shape)` {#Exponential.param_static_shapes}

param_shapes with static (i.e. TensorShape) shapes.

##### Args:


*  <b>`sample_shape`</b>: `TensorShape` or python list/tuple. Desired shape of a call
    to `sample()`.

##### Returns:

  `dict` of parameter name to `TensorShape`.

##### Raises:


*  <b>`ValueError`</b>: if `sample_shape` is a `TensorShape` and is not fully defined.


- - -

#### `tf.contrib.distributions.Exponential.parameters` {#Exponential.parameters}

Dictionary of parameters used by this `Distribution`.


- - -

#### `tf.contrib.distributions.Exponential.pdf(value, name='pdf')` {#Exponential.pdf}

Probability density function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if not `is_continuous`.


- - -

#### `tf.contrib.distributions.Exponential.pmf(value, name='pmf')` {#Exponential.pmf}

Probability mass function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`pmf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if `is_continuous`.


- - -

#### `tf.contrib.distributions.Exponential.prob(value, name='prob')` {#Exponential.prob}

Probability density/mass function (depending on `is_continuous`).

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Exponential.sample(sample_shape=(), seed=None, name='sample')` {#Exponential.sample}

Generate samples of the specified shape.

Note that a call to `sample()` without arguments will generate a single
sample.

##### Args:


*  <b>`sample_shape`</b>: 0D or 1D `int32` `Tensor`. Shape of the generated samples.
*  <b>`seed`</b>: Python integer seed for RNG
*  <b>`name`</b>: name to give to the op.

##### Returns:


*  <b>`samples`</b>: a `Tensor` with prepended dimensions `sample_shape`.


- - -

#### `tf.contrib.distributions.Exponential.sample_n(n, seed=None, name='sample_n')` {#Exponential.sample_n}

Generate `n` samples.

##### Args:


*  <b>`n`</b>: `Scalar` `Tensor` of type `int32` or `int64`, the number of
    observations to sample.
*  <b>`seed`</b>: Python integer seed for RNG
*  <b>`name`</b>: name to give to the op.

##### Returns:


*  <b>`samples`</b>: a `Tensor` with a prepended dimension (n,).

##### Raises:


*  <b>`TypeError`</b>: if `n` is not an integer type.


- - -

#### `tf.contrib.distributions.Exponential.std(name='std')` {#Exponential.std}

Standard deviation.


- - -

#### `tf.contrib.distributions.Exponential.validate_args` {#Exponential.validate_args}

Python boolean indicated possibly expensive checks are enabled.


- - -

#### `tf.contrib.distributions.Exponential.variance(name='variance')` {#Exponential.variance}

Variance.



- - -

### `class tf.contrib.distributions.Gamma` {#Gamma}

The `Gamma` distribution with parameter alpha and beta.

The parameters are the shape and inverse scale parameters alpha, beta.

The PDF of this distribution is:

```pdf(x) = (beta^alpha)(x^(alpha-1))e^(-x*beta)/Gamma(alpha), x > 0```

and the CDF of this distribution is:

```cdf(x) =  GammaInc(alpha, beta * x) / Gamma(alpha), x > 0```

where GammaInc is the incomplete lower Gamma function.

WARNING: This distribution may draw 0-valued samples for small alpha values.
    See the note on `tf.random_gamma`.

Examples:

```python
dist = Gamma(alpha=3.0, beta=2.0)
dist2 = Gamma(alpha=[3.0, 4.0], beta=[2.0, 3.0])
```
- - -

#### `tf.contrib.distributions.Gamma.__init__(alpha, beta, validate_args=True, allow_nan_stats=False, name='Gamma')` {#Gamma.__init__}

Construct Gamma distributions with parameters `alpha` and `beta`.

The parameters `alpha` and `beta` must be shaped in a way that supports
broadcasting (e.g. `alpha + beta` is a valid operation).

##### Args:


*  <b>`alpha`</b>: Floating point tensor, the shape params of the
    distribution(s).
    alpha must contain only positive values.
*  <b>`beta`</b>: Floating point tensor, the inverse scale params of the
    distribution(s).
    beta must contain only positive values.
*  <b>`validate_args`</b>: Whether to assert that `a > 0, b > 0`, and that `x > 0` in
    the methods `prob(x)` and `log_prob(x)`.  If `validate_args` is `False`
    and the inputs are invalid, correct behavior is not guaranteed.
*  <b>`allow_nan_stats`</b>: Boolean, default `False`.  If `False`, raise an
    exception if a statistic (e.g. mean/mode/etc...) is undefined for any
    batch member.  If `True`, batch members with valid parameters leading to
    undefined statistics will return NaN for this statistic.
*  <b>`name`</b>: The name to prepend to all ops created by this distribution.

##### Raises:


*  <b>`TypeError`</b>: if `alpha` and `beta` are different dtypes.


- - -

#### `tf.contrib.distributions.Gamma.allow_nan_stats` {#Gamma.allow_nan_stats}

Python boolean describing behavior when a stat is undefined.

Stats return +/- infinity when it makes sense.  E.g., the variance
of a Cauchy distribution is infinity.  However, sometimes the
statistic is undefined, e.g., if a distribution's pdf does not achieve a
maximum within the support of the distribution, the mode is undefined.
If the mean is undefined, then by definition the variance is undefined.
E.g. the mean for Student's T for df = 1 is undefined (no clear way to say
it is either + or - infinity), so the variance = E[(X - mean)^2] is also
undefined.

##### Returns:


*  <b>`allow_nan_stats`</b>: Python boolean.


- - -

#### `tf.contrib.distributions.Gamma.alpha` {#Gamma.alpha}

Shape parameter.


- - -

#### `tf.contrib.distributions.Gamma.batch_shape(name='batch_shape')` {#Gamma.batch_shape}

Shape of a single sample from a single event index as a 1-D `Tensor`.

The product of the dimensions of the `batch_shape` is the number of
independent distributions of this kind the instance represents.

##### Args:


*  <b>`name`</b>: name to give to the op

##### Returns:


*  <b>`batch_shape`</b>: `Tensor`.


- - -

#### `tf.contrib.distributions.Gamma.beta` {#Gamma.beta}

Inverse scale parameter.


- - -

#### `tf.contrib.distributions.Gamma.cdf(value, name='cdf')` {#Gamma.cdf}

Cumulative distribution function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`cdf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Gamma.dtype` {#Gamma.dtype}

The `DType` of `Tensor`s handled by this `Distribution`.


- - -

#### `tf.contrib.distributions.Gamma.entropy(name='entropy')` {#Gamma.entropy}

Shanon entropy in nats.


- - -

#### `tf.contrib.distributions.Gamma.event_shape(name='event_shape')` {#Gamma.event_shape}

Shape of a single sample from a single batch as a 1-D int32 `Tensor`.

##### Args:


*  <b>`name`</b>: name to give to the op

##### Returns:


*  <b>`event_shape`</b>: `Tensor`.


- - -

#### `tf.contrib.distributions.Gamma.from_params(cls, make_safe=True, **kwargs)` {#Gamma.from_params}

Given (unconstrained) parameters, return an instantiated distribution.

Subclasses should implement a static method `_safe_transforms` that returns
a dict of parameter transforms, which will be used if `make_safe = True`.

Example usage:

```
# Let's say we want a sample of size (batch_size, 10)
shapes = MultiVariateNormalDiag.param_shapes([batch_size, 10])

# shapes has a Tensor shape for mu and sigma
# shapes == {
#   "mu": tf.constant([batch_size, 10]),
#   "sigma": tf.constant([batch_size, 10]),
# }

# Here we parameterize mu and sigma with the output of a linear
# layer. Note that sigma is unconstrained.
params = {}
for name, shape in shapes.items():
  params[name] = linear(x, shape[1])

# Note that you can forward other kwargs to the `Distribution`, like
# `allow_nan_stats` or `name`.
mvn = MultiVariateNormalDiag.from_params(**params, allow_nan_stats=True)
```

Distribution parameters may have constraints (e.g. `sigma` must be positive
for a `Normal` distribution) and the `from_params` method will apply default
parameter transforms. If a user wants to use their own transform, they can
apply it externally and set `make_safe=False`.

##### Args:


*  <b>`make_safe`</b>: Whether the `params` should be constrained. If True,
    `from_params` will apply default parameter transforms. If False, no
    parameter transforms will be applied.
*  <b>`**kwargs`</b>: dict of parameters for the distribution.

##### Returns:

  A distribution parameterized by possibly transformed parameters in
  `kwargs`.

##### Raises:


*  <b>`TypeError`</b>: if `make_safe` is `True` but `_safe_transforms` is not
    implemented directly for `cls`.


- - -

#### `tf.contrib.distributions.Gamma.get_batch_shape()` {#Gamma.get_batch_shape}

Shape of a single sample from a single event index as a `TensorShape`.

Same meaning as `batch_shape`. May be only partially defined.

##### Returns:


*  <b>`batch_shape`</b>: `TensorShape`, possibly unknown.


- - -

#### `tf.contrib.distributions.Gamma.get_event_shape()` {#Gamma.get_event_shape}

Shape of a single sample from a single batch as a `TensorShape`.

Same meaning as `event_shape`. May be only partially defined.

##### Returns:


*  <b>`event_shape`</b>: `TensorShape`, possibly unknown.


- - -

#### `tf.contrib.distributions.Gamma.is_continuous` {#Gamma.is_continuous}




- - -

#### `tf.contrib.distributions.Gamma.is_reparameterized` {#Gamma.is_reparameterized}




- - -

#### `tf.contrib.distributions.Gamma.log_cdf(value, name='log_cdf')` {#Gamma.log_cdf}

Log cumulative distribution function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`logcdf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Gamma.log_pdf(value, name='log_pdf')` {#Gamma.log_pdf}

Log probability density function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if not `is_continuous`.


- - -

#### `tf.contrib.distributions.Gamma.log_pmf(value, name='log_pmf')` {#Gamma.log_pmf}

Log probability mass function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_pmf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if `is_continuous`.


- - -

#### `tf.contrib.distributions.Gamma.log_prob(value, name='log_prob')` {#Gamma.log_prob}

Log probability density/mass function (depending on `is_continuous`).

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Gamma.mean(name='mean')` {#Gamma.mean}

Mean.


- - -

#### `tf.contrib.distributions.Gamma.mode(name='mode')` {#Gamma.mode}

Mode.


- - -

#### `tf.contrib.distributions.Gamma.name` {#Gamma.name}

Name prepended to all ops created by this `Distribution`.


- - -

#### `tf.contrib.distributions.Gamma.param_shapes(cls, sample_shape, name='DistributionParamShapes')` {#Gamma.param_shapes}

Shapes of parameters given the desired shape of a call to `sample()`.

Subclasses should override static method `_param_shapes`.

##### Args:


*  <b>`sample_shape`</b>: `Tensor` or python list/tuple. Desired shape of a call to
    `sample()`.
*  <b>`name`</b>: name to prepend ops with.

##### Returns:

  `dict` of parameter name to `Tensor` shapes.


- - -

#### `tf.contrib.distributions.Gamma.param_static_shapes(cls, sample_shape)` {#Gamma.param_static_shapes}

param_shapes with static (i.e. TensorShape) shapes.

##### Args:


*  <b>`sample_shape`</b>: `TensorShape` or python list/tuple. Desired shape of a call
    to `sample()`.

##### Returns:

  `dict` of parameter name to `TensorShape`.

##### Raises:


*  <b>`ValueError`</b>: if `sample_shape` is a `TensorShape` and is not fully defined.


- - -

#### `tf.contrib.distributions.Gamma.parameters` {#Gamma.parameters}

Dictionary of parameters used by this `Distribution`.


- - -

#### `tf.contrib.distributions.Gamma.pdf(value, name='pdf')` {#Gamma.pdf}

Probability density function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if not `is_continuous`.


- - -

#### `tf.contrib.distributions.Gamma.pmf(value, name='pmf')` {#Gamma.pmf}

Probability mass function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`pmf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if `is_continuous`.


- - -

#### `tf.contrib.distributions.Gamma.prob(value, name='prob')` {#Gamma.prob}

Probability density/mass function (depending on `is_continuous`).

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Gamma.sample(sample_shape=(), seed=None, name='sample')` {#Gamma.sample}

Generate samples of the specified shape.

Note that a call to `sample()` without arguments will generate a single
sample.

##### Args:


*  <b>`sample_shape`</b>: 0D or 1D `int32` `Tensor`. Shape of the generated samples.
*  <b>`seed`</b>: Python integer seed for RNG
*  <b>`name`</b>: name to give to the op.

##### Returns:


*  <b>`samples`</b>: a `Tensor` with prepended dimensions `sample_shape`.


- - -

#### `tf.contrib.distributions.Gamma.sample_n(n, seed=None, name='sample_n')` {#Gamma.sample_n}

Generate `n` samples.

##### Args:


*  <b>`n`</b>: `Scalar` `Tensor` of type `int32` or `int64`, the number of
    observations to sample.
*  <b>`seed`</b>: Python integer seed for RNG
*  <b>`name`</b>: name to give to the op.

##### Returns:


*  <b>`samples`</b>: a `Tensor` with a prepended dimension (n,).

##### Raises:


*  <b>`TypeError`</b>: if `n` is not an integer type.


- - -

#### `tf.contrib.distributions.Gamma.std(name='std')` {#Gamma.std}

Standard deviation.


- - -

#### `tf.contrib.distributions.Gamma.validate_args` {#Gamma.validate_args}

Python boolean indicated possibly expensive checks are enabled.


- - -

#### `tf.contrib.distributions.Gamma.variance(name='variance')` {#Gamma.variance}

Variance.



- - -

### `class tf.contrib.distributions.InverseGamma` {#InverseGamma}

The `InverseGamma` distribution with parameter alpha and beta.

The parameters are the shape and inverse scale parameters alpha, beta.

The PDF of this distribution is:

```pdf(x) = (beta^alpha)/Gamma(alpha)(x^(-alpha-1))e^(-beta/x), x > 0```

and the CDF of this distribution is:

```cdf(x) =  GammaInc(alpha, beta / x) / Gamma(alpha), x > 0```

where GammaInc is the upper incomplete Gamma function.

Examples:

```python
dist = InverseGamma(alpha=3.0, beta=2.0)
dist2 = InverseGamma(alpha=[3.0, 4.0], beta=[2.0, 3.0])
```
- - -

#### `tf.contrib.distributions.InverseGamma.__init__(alpha, beta, validate_args=True, allow_nan_stats=False, name='InverseGamma')` {#InverseGamma.__init__}

Construct InverseGamma distributions with parameters `alpha` and `beta`.

The parameters `alpha` and `beta` must be shaped in a way that supports
broadcasting (e.g. `alpha + beta` is a valid operation).

##### Args:


*  <b>`alpha`</b>: Floating point tensor, the shape params of the
    distribution(s).
    alpha must contain only positive values.
*  <b>`beta`</b>: Floating point tensor, the scale params of the distribution(s).
    beta must contain only positive values.
*  <b>`validate_args`</b>: Whether to assert that `a > 0, b > 0`, and that `x > 0` in
    the methods `prob(x)` and `log_prob(x)`.  If `validate_args` is `False`
    and the inputs are invalid, correct behavior is not guaranteed.
*  <b>`allow_nan_stats`</b>: Boolean, default `False`.  If `False`, raise an
    exception if a statistic (e.g. mean/mode/etc...) is undefined for any
    batch member.  If `True`, batch members with valid parameters leading to
    undefined statistics will return NaN for this statistic.
*  <b>`name`</b>: The name to prepend to all ops created by this distribution.

##### Raises:


*  <b>`TypeError`</b>: if `alpha` and `beta` are different dtypes.


- - -

#### `tf.contrib.distributions.InverseGamma.allow_nan_stats` {#InverseGamma.allow_nan_stats}

Python boolean describing behavior when a stat is undefined.

Stats return +/- infinity when it makes sense.  E.g., the variance
of a Cauchy distribution is infinity.  However, sometimes the
statistic is undefined, e.g., if a distribution's pdf does not achieve a
maximum within the support of the distribution, the mode is undefined.
If the mean is undefined, then by definition the variance is undefined.
E.g. the mean for Student's T for df = 1 is undefined (no clear way to say
it is either + or - infinity), so the variance = E[(X - mean)^2] is also
undefined.

##### Returns:


*  <b>`allow_nan_stats`</b>: Python boolean.


- - -

#### `tf.contrib.distributions.InverseGamma.alpha` {#InverseGamma.alpha}

Shape parameter.


- - -

#### `tf.contrib.distributions.InverseGamma.batch_shape(name='batch_shape')` {#InverseGamma.batch_shape}

Shape of a single sample from a single event index as a 1-D `Tensor`.

The product of the dimensions of the `batch_shape` is the number of
independent distributions of this kind the instance represents.

##### Args:


*  <b>`name`</b>: name to give to the op

##### Returns:


*  <b>`batch_shape`</b>: `Tensor`.


- - -

#### `tf.contrib.distributions.InverseGamma.beta` {#InverseGamma.beta}

Scale parameter.


- - -

#### `tf.contrib.distributions.InverseGamma.cdf(value, name='cdf')` {#InverseGamma.cdf}

Cumulative distribution function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`cdf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.InverseGamma.dtype` {#InverseGamma.dtype}

The `DType` of `Tensor`s handled by this `Distribution`.


- - -

#### `tf.contrib.distributions.InverseGamma.entropy(name='entropy')` {#InverseGamma.entropy}

Shanon entropy in nats.


- - -

#### `tf.contrib.distributions.InverseGamma.event_shape(name='event_shape')` {#InverseGamma.event_shape}

Shape of a single sample from a single batch as a 1-D int32 `Tensor`.

##### Args:


*  <b>`name`</b>: name to give to the op

##### Returns:


*  <b>`event_shape`</b>: `Tensor`.


- - -

#### `tf.contrib.distributions.InverseGamma.from_params(cls, make_safe=True, **kwargs)` {#InverseGamma.from_params}

Given (unconstrained) parameters, return an instantiated distribution.

Subclasses should implement a static method `_safe_transforms` that returns
a dict of parameter transforms, which will be used if `make_safe = True`.

Example usage:

```
# Let's say we want a sample of size (batch_size, 10)
shapes = MultiVariateNormalDiag.param_shapes([batch_size, 10])

# shapes has a Tensor shape for mu and sigma
# shapes == {
#   "mu": tf.constant([batch_size, 10]),
#   "sigma": tf.constant([batch_size, 10]),
# }

# Here we parameterize mu and sigma with the output of a linear
# layer. Note that sigma is unconstrained.
params = {}
for name, shape in shapes.items():
  params[name] = linear(x, shape[1])

# Note that you can forward other kwargs to the `Distribution`, like
# `allow_nan_stats` or `name`.
mvn = MultiVariateNormalDiag.from_params(**params, allow_nan_stats=True)
```

Distribution parameters may have constraints (e.g. `sigma` must be positive
for a `Normal` distribution) and the `from_params` method will apply default
parameter transforms. If a user wants to use their own transform, they can
apply it externally and set `make_safe=False`.

##### Args:


*  <b>`make_safe`</b>: Whether the `params` should be constrained. If True,
    `from_params` will apply default parameter transforms. If False, no
    parameter transforms will be applied.
*  <b>`**kwargs`</b>: dict of parameters for the distribution.

##### Returns:

  A distribution parameterized by possibly transformed parameters in
  `kwargs`.

##### Raises:


*  <b>`TypeError`</b>: if `make_safe` is `True` but `_safe_transforms` is not
    implemented directly for `cls`.


- - -

#### `tf.contrib.distributions.InverseGamma.get_batch_shape()` {#InverseGamma.get_batch_shape}

Shape of a single sample from a single event index as a `TensorShape`.

Same meaning as `batch_shape`. May be only partially defined.

##### Returns:


*  <b>`batch_shape`</b>: `TensorShape`, possibly unknown.


- - -

#### `tf.contrib.distributions.InverseGamma.get_event_shape()` {#InverseGamma.get_event_shape}

Shape of a single sample from a single batch as a `TensorShape`.

Same meaning as `event_shape`. May be only partially defined.

##### Returns:


*  <b>`event_shape`</b>: `TensorShape`, possibly unknown.


- - -

#### `tf.contrib.distributions.InverseGamma.is_continuous` {#InverseGamma.is_continuous}




- - -

#### `tf.contrib.distributions.InverseGamma.is_reparameterized` {#InverseGamma.is_reparameterized}




- - -

#### `tf.contrib.distributions.InverseGamma.log_cdf(value, name='log_cdf')` {#InverseGamma.log_cdf}

Log cumulative distribution function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`logcdf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.InverseGamma.log_pdf(value, name='log_pdf')` {#InverseGamma.log_pdf}

Log probability density function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if not `is_continuous`.


- - -

#### `tf.contrib.distributions.InverseGamma.log_pmf(value, name='log_pmf')` {#InverseGamma.log_pmf}

Log probability mass function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_pmf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if `is_continuous`.


- - -

#### `tf.contrib.distributions.InverseGamma.log_prob(value, name='log_prob')` {#InverseGamma.log_prob}

Log probability density/mass function (depending on `is_continuous`).

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.InverseGamma.mean(name='mean')` {#InverseGamma.mean}

Mean.


- - -

#### `tf.contrib.distributions.InverseGamma.mode(name='mode')` {#InverseGamma.mode}

Mode.


- - -

#### `tf.contrib.distributions.InverseGamma.name` {#InverseGamma.name}

Name prepended to all ops created by this `Distribution`.


- - -

#### `tf.contrib.distributions.InverseGamma.param_shapes(cls, sample_shape, name='DistributionParamShapes')` {#InverseGamma.param_shapes}

Shapes of parameters given the desired shape of a call to `sample()`.

Subclasses should override static method `_param_shapes`.

##### Args:


*  <b>`sample_shape`</b>: `Tensor` or python list/tuple. Desired shape of a call to
    `sample()`.
*  <b>`name`</b>: name to prepend ops with.

##### Returns:

  `dict` of parameter name to `Tensor` shapes.


- - -

#### `tf.contrib.distributions.InverseGamma.param_static_shapes(cls, sample_shape)` {#InverseGamma.param_static_shapes}

param_shapes with static (i.e. TensorShape) shapes.

##### Args:


*  <b>`sample_shape`</b>: `TensorShape` or python list/tuple. Desired shape of a call
    to `sample()`.

##### Returns:

  `dict` of parameter name to `TensorShape`.

##### Raises:


*  <b>`ValueError`</b>: if `sample_shape` is a `TensorShape` and is not fully defined.


- - -

#### `tf.contrib.distributions.InverseGamma.parameters` {#InverseGamma.parameters}

Dictionary of parameters used by this `Distribution`.


- - -

#### `tf.contrib.distributions.InverseGamma.pdf(value, name='pdf')` {#InverseGamma.pdf}

Probability density function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if not `is_continuous`.


- - -

#### `tf.contrib.distributions.InverseGamma.pmf(value, name='pmf')` {#InverseGamma.pmf}

Probability mass function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`pmf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if `is_continuous`.


- - -

#### `tf.contrib.distributions.InverseGamma.prob(value, name='prob')` {#InverseGamma.prob}

Probability density/mass function (depending on `is_continuous`).

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.InverseGamma.sample(sample_shape=(), seed=None, name='sample')` {#InverseGamma.sample}

Generate samples of the specified shape.

Note that a call to `sample()` without arguments will generate a single
sample.

##### Args:


*  <b>`sample_shape`</b>: 0D or 1D `int32` `Tensor`. Shape of the generated samples.
*  <b>`seed`</b>: Python integer seed for RNG
*  <b>`name`</b>: name to give to the op.

##### Returns:


*  <b>`samples`</b>: a `Tensor` with prepended dimensions `sample_shape`.


- - -

#### `tf.contrib.distributions.InverseGamma.sample_n(n, seed=None, name='sample_n')` {#InverseGamma.sample_n}

Generate `n` samples.

##### Args:


*  <b>`n`</b>: `Scalar` `Tensor` of type `int32` or `int64`, the number of
    observations to sample.
*  <b>`seed`</b>: Python integer seed for RNG
*  <b>`name`</b>: name to give to the op.

##### Returns:


*  <b>`samples`</b>: a `Tensor` with a prepended dimension (n,).

##### Raises:


*  <b>`TypeError`</b>: if `n` is not an integer type.


- - -

#### `tf.contrib.distributions.InverseGamma.std(name='std')` {#InverseGamma.std}

Standard deviation.


- - -

#### `tf.contrib.distributions.InverseGamma.validate_args` {#InverseGamma.validate_args}

Python boolean indicated possibly expensive checks are enabled.


- - -

#### `tf.contrib.distributions.InverseGamma.variance(name='variance')` {#InverseGamma.variance}

Variance.



- - -

### `class tf.contrib.distributions.Laplace` {#Laplace}

The Laplace distribution with location and scale > 0 parameters.

#### Mathematical details

The PDF of this distribution is:

```f(x | mu, b, b > 0) = 0.5 / b exp(-|x - mu| / b)```

Note that the Laplace distribution can be thought of two exponential
distributions spliced together "back-to-back."
- - -

#### `tf.contrib.distributions.Laplace.__init__(loc, scale, validate_args=True, allow_nan_stats=False, name='Laplace')` {#Laplace.__init__}

Construct Laplace distribution with parameters `loc` and `scale`.

The parameters `loc` and `scale` must be shaped in a way that supports
broadcasting (e.g., `loc / scale` is a valid operation).

##### Args:


*  <b>`loc`</b>: Floating point tensor which characterizes the location (center)
    of the distribution.
*  <b>`scale`</b>: Positive floating point tensor which characterizes the spread of
    the distribution.
*  <b>`validate_args`</b>: Whether to validate input with asserts.  If `validate_args`
    is `False`, and the inputs are invalid, correct behavior is not
    guaranteed.
*  <b>`allow_nan_stats`</b>: Boolean, default `False`.  If `False`, raise an
    exception if a statistic (e.g. mean/mode/etc...) is undefined for any
    batch member.  If `True`, batch members with valid parameters leading to
    undefined statistics will return NaN for this statistic.
*  <b>`name`</b>: The name to give Ops created by the initializer.

##### Raises:


*  <b>`TypeError`</b>: if `loc` and `scale` are of different dtype.


- - -

#### `tf.contrib.distributions.Laplace.allow_nan_stats` {#Laplace.allow_nan_stats}

Python boolean describing behavior when a stat is undefined.

Stats return +/- infinity when it makes sense.  E.g., the variance
of a Cauchy distribution is infinity.  However, sometimes the
statistic is undefined, e.g., if a distribution's pdf does not achieve a
maximum within the support of the distribution, the mode is undefined.
If the mean is undefined, then by definition the variance is undefined.
E.g. the mean for Student's T for df = 1 is undefined (no clear way to say
it is either + or - infinity), so the variance = E[(X - mean)^2] is also
undefined.

##### Returns:


*  <b>`allow_nan_stats`</b>: Python boolean.


- - -

#### `tf.contrib.distributions.Laplace.batch_shape(name='batch_shape')` {#Laplace.batch_shape}

Shape of a single sample from a single event index as a 1-D `Tensor`.

The product of the dimensions of the `batch_shape` is the number of
independent distributions of this kind the instance represents.

##### Args:


*  <b>`name`</b>: name to give to the op

##### Returns:


*  <b>`batch_shape`</b>: `Tensor`.


- - -

#### `tf.contrib.distributions.Laplace.cdf(value, name='cdf')` {#Laplace.cdf}

Cumulative distribution function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`cdf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Laplace.dtype` {#Laplace.dtype}

The `DType` of `Tensor`s handled by this `Distribution`.


- - -

#### `tf.contrib.distributions.Laplace.entropy(name='entropy')` {#Laplace.entropy}

Shanon entropy in nats.


- - -

#### `tf.contrib.distributions.Laplace.event_shape(name='event_shape')` {#Laplace.event_shape}

Shape of a single sample from a single batch as a 1-D int32 `Tensor`.

##### Args:


*  <b>`name`</b>: name to give to the op

##### Returns:


*  <b>`event_shape`</b>: `Tensor`.


- - -

#### `tf.contrib.distributions.Laplace.from_params(cls, make_safe=True, **kwargs)` {#Laplace.from_params}

Given (unconstrained) parameters, return an instantiated distribution.

Subclasses should implement a static method `_safe_transforms` that returns
a dict of parameter transforms, which will be used if `make_safe = True`.

Example usage:

```
# Let's say we want a sample of size (batch_size, 10)
shapes = MultiVariateNormalDiag.param_shapes([batch_size, 10])

# shapes has a Tensor shape for mu and sigma
# shapes == {
#   "mu": tf.constant([batch_size, 10]),
#   "sigma": tf.constant([batch_size, 10]),
# }

# Here we parameterize mu and sigma with the output of a linear
# layer. Note that sigma is unconstrained.
params = {}
for name, shape in shapes.items():
  params[name] = linear(x, shape[1])

# Note that you can forward other kwargs to the `Distribution`, like
# `allow_nan_stats` or `name`.
mvn = MultiVariateNormalDiag.from_params(**params, allow_nan_stats=True)
```

Distribution parameters may have constraints (e.g. `sigma` must be positive
for a `Normal` distribution) and the `from_params` method will apply default
parameter transforms. If a user wants to use their own transform, they can
apply it externally and set `make_safe=False`.

##### Args:


*  <b>`make_safe`</b>: Whether the `params` should be constrained. If True,
    `from_params` will apply default parameter transforms. If False, no
    parameter transforms will be applied.
*  <b>`**kwargs`</b>: dict of parameters for the distribution.

##### Returns:

  A distribution parameterized by possibly transformed parameters in
  `kwargs`.

##### Raises:


*  <b>`TypeError`</b>: if `make_safe` is `True` but `_safe_transforms` is not
    implemented directly for `cls`.


- - -

#### `tf.contrib.distributions.Laplace.get_batch_shape()` {#Laplace.get_batch_shape}

Shape of a single sample from a single event index as a `TensorShape`.

Same meaning as `batch_shape`. May be only partially defined.

##### Returns:


*  <b>`batch_shape`</b>: `TensorShape`, possibly unknown.


- - -

#### `tf.contrib.distributions.Laplace.get_event_shape()` {#Laplace.get_event_shape}

Shape of a single sample from a single batch as a `TensorShape`.

Same meaning as `event_shape`. May be only partially defined.

##### Returns:


*  <b>`event_shape`</b>: `TensorShape`, possibly unknown.


- - -

#### `tf.contrib.distributions.Laplace.is_continuous` {#Laplace.is_continuous}




- - -

#### `tf.contrib.distributions.Laplace.is_reparameterized` {#Laplace.is_reparameterized}




- - -

#### `tf.contrib.distributions.Laplace.loc` {#Laplace.loc}

Distribution parameter for the location.


- - -

#### `tf.contrib.distributions.Laplace.log_cdf(value, name='log_cdf')` {#Laplace.log_cdf}

Log cumulative distribution function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`logcdf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Laplace.log_pdf(value, name='log_pdf')` {#Laplace.log_pdf}

Log probability density function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if not `is_continuous`.


- - -

#### `tf.contrib.distributions.Laplace.log_pmf(value, name='log_pmf')` {#Laplace.log_pmf}

Log probability mass function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_pmf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if `is_continuous`.


- - -

#### `tf.contrib.distributions.Laplace.log_prob(value, name='log_prob')` {#Laplace.log_prob}

Log probability density/mass function (depending on `is_continuous`).

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Laplace.mean(name='mean')` {#Laplace.mean}

Mean.


- - -

#### `tf.contrib.distributions.Laplace.mode(name='mode')` {#Laplace.mode}

Mode.


- - -

#### `tf.contrib.distributions.Laplace.name` {#Laplace.name}

Name prepended to all ops created by this `Distribution`.


- - -

#### `tf.contrib.distributions.Laplace.param_shapes(cls, sample_shape, name='DistributionParamShapes')` {#Laplace.param_shapes}

Shapes of parameters given the desired shape of a call to `sample()`.

Subclasses should override static method `_param_shapes`.

##### Args:


*  <b>`sample_shape`</b>: `Tensor` or python list/tuple. Desired shape of a call to
    `sample()`.
*  <b>`name`</b>: name to prepend ops with.

##### Returns:

  `dict` of parameter name to `Tensor` shapes.


- - -

#### `tf.contrib.distributions.Laplace.param_static_shapes(cls, sample_shape)` {#Laplace.param_static_shapes}

param_shapes with static (i.e. TensorShape) shapes.

##### Args:


*  <b>`sample_shape`</b>: `TensorShape` or python list/tuple. Desired shape of a call
    to `sample()`.

##### Returns:

  `dict` of parameter name to `TensorShape`.

##### Raises:


*  <b>`ValueError`</b>: if `sample_shape` is a `TensorShape` and is not fully defined.


- - -

#### `tf.contrib.distributions.Laplace.parameters` {#Laplace.parameters}

Dictionary of parameters used by this `Distribution`.


- - -

#### `tf.contrib.distributions.Laplace.pdf(value, name='pdf')` {#Laplace.pdf}

Probability density function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if not `is_continuous`.


- - -

#### `tf.contrib.distributions.Laplace.pmf(value, name='pmf')` {#Laplace.pmf}

Probability mass function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`pmf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if `is_continuous`.


- - -

#### `tf.contrib.distributions.Laplace.prob(value, name='prob')` {#Laplace.prob}

Probability density/mass function (depending on `is_continuous`).

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Laplace.sample(sample_shape=(), seed=None, name='sample')` {#Laplace.sample}

Generate samples of the specified shape.

Note that a call to `sample()` without arguments will generate a single
sample.

##### Args:


*  <b>`sample_shape`</b>: 0D or 1D `int32` `Tensor`. Shape of the generated samples.
*  <b>`seed`</b>: Python integer seed for RNG
*  <b>`name`</b>: name to give to the op.

##### Returns:


*  <b>`samples`</b>: a `Tensor` with prepended dimensions `sample_shape`.


- - -

#### `tf.contrib.distributions.Laplace.sample_n(n, seed=None, name='sample_n')` {#Laplace.sample_n}

Generate `n` samples.

##### Args:


*  <b>`n`</b>: `Scalar` `Tensor` of type `int32` or `int64`, the number of
    observations to sample.
*  <b>`seed`</b>: Python integer seed for RNG
*  <b>`name`</b>: name to give to the op.

##### Returns:


*  <b>`samples`</b>: a `Tensor` with a prepended dimension (n,).

##### Raises:


*  <b>`TypeError`</b>: if `n` is not an integer type.


- - -

#### `tf.contrib.distributions.Laplace.scale` {#Laplace.scale}

Distribution parameter for scale.


- - -

#### `tf.contrib.distributions.Laplace.std(name='std')` {#Laplace.std}

Standard deviation.


- - -

#### `tf.contrib.distributions.Laplace.validate_args` {#Laplace.validate_args}

Python boolean indicated possibly expensive checks are enabled.


- - -

#### `tf.contrib.distributions.Laplace.variance(name='variance')` {#Laplace.variance}

Variance.



- - -

### `class tf.contrib.distributions.Normal` {#Normal}

The scalar Normal distribution with mean and stddev parameters mu, sigma.

#### Mathematical details

The PDF of this distribution is:

```f(x) = sqrt(1/(2*pi*sigma^2)) exp(-(x-mu)^2/(2*sigma^2))```

#### Examples

Examples of initialization of one or a batch of distributions.

```python
# Define a single scalar Normal distribution.
dist = tf.contrib.distributions.Normal(mu=0., sigma=3.)

# Evaluate the cdf at 1, returning a scalar.
dist.cdf(1.)

# Define a batch of two scalar valued Normals.
# The first has mean 1 and standard deviation 11, the second 2 and 22.
dist = tf.contrib.distributions.Normal(mu=[1, 2.], sigma=[11, 22.])

# Evaluate the pdf of the first distribution on 0, and the second on 1.5,
# returning a length two tensor.
dist.pdf([0, 1.5])

# Get 3 samples, returning a 3 x 2 tensor.
dist.sample([3])
```

Arguments are broadcast when possible.

```python
# Define a batch of two scalar valued Normals.
# Both have mean 1, but different standard deviations.
dist = tf.contrib.distributions.Normal(mu=1., sigma=[11, 22.])

# Evaluate the pdf of both distributions on the same point, 3.0,
# returning a length 2 tensor.
dist.pdf(3.0)
```
- - -

#### `tf.contrib.distributions.Normal.__init__(mu, sigma, validate_args=True, allow_nan_stats=False, name='Normal')` {#Normal.__init__}

Construct Normal distributions with mean and stddev `mu` and `sigma`.

The parameters `mu` and `sigma` must be shaped in a way that supports
broadcasting (e.g. `mu + sigma` is a valid operation).

##### Args:


*  <b>`mu`</b>: Floating point tensor, the means of the distribution(s).
*  <b>`sigma`</b>: Floating point tensor, the stddevs of the distribution(s).
    sigma must contain only positive values.
*  <b>`validate_args`</b>: Whether to assert that `sigma > 0`. If `validate_args` is
    `False`, correct output is not guaranteed when input is invalid.
*  <b>`allow_nan_stats`</b>: Boolean, default `False`.  If `False`, raise an
    exception if a statistic (e.g. mean/mode/etc...) is undefined for any
    batch member.  If `True`, batch members with valid parameters leading to
    undefined statistics will return NaN for this statistic.
*  <b>`name`</b>: The name to give Ops created by the initializer.

##### Raises:


*  <b>`TypeError`</b>: if mu and sigma are different dtypes.


- - -

#### `tf.contrib.distributions.Normal.allow_nan_stats` {#Normal.allow_nan_stats}

Python boolean describing behavior when a stat is undefined.

Stats return +/- infinity when it makes sense.  E.g., the variance
of a Cauchy distribution is infinity.  However, sometimes the
statistic is undefined, e.g., if a distribution's pdf does not achieve a
maximum within the support of the distribution, the mode is undefined.
If the mean is undefined, then by definition the variance is undefined.
E.g. the mean for Student's T for df = 1 is undefined (no clear way to say
it is either + or - infinity), so the variance = E[(X - mean)^2] is also
undefined.

##### Returns:


*  <b>`allow_nan_stats`</b>: Python boolean.


- - -

#### `tf.contrib.distributions.Normal.batch_shape(name='batch_shape')` {#Normal.batch_shape}

Shape of a single sample from a single event index as a 1-D `Tensor`.

The product of the dimensions of the `batch_shape` is the number of
independent distributions of this kind the instance represents.

##### Args:


*  <b>`name`</b>: name to give to the op

##### Returns:


*  <b>`batch_shape`</b>: `Tensor`.


- - -

#### `tf.contrib.distributions.Normal.cdf(value, name='cdf')` {#Normal.cdf}

Cumulative distribution function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`cdf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Normal.dtype` {#Normal.dtype}

The `DType` of `Tensor`s handled by this `Distribution`.


- - -

#### `tf.contrib.distributions.Normal.entropy(name='entropy')` {#Normal.entropy}

Shanon entropy in nats.


- - -

#### `tf.contrib.distributions.Normal.event_shape(name='event_shape')` {#Normal.event_shape}

Shape of a single sample from a single batch as a 1-D int32 `Tensor`.

##### Args:


*  <b>`name`</b>: name to give to the op

##### Returns:


*  <b>`event_shape`</b>: `Tensor`.


- - -

#### `tf.contrib.distributions.Normal.from_params(cls, make_safe=True, **kwargs)` {#Normal.from_params}

Given (unconstrained) parameters, return an instantiated distribution.

Subclasses should implement a static method `_safe_transforms` that returns
a dict of parameter transforms, which will be used if `make_safe = True`.

Example usage:

```
# Let's say we want a sample of size (batch_size, 10)
shapes = MultiVariateNormalDiag.param_shapes([batch_size, 10])

# shapes has a Tensor shape for mu and sigma
# shapes == {
#   "mu": tf.constant([batch_size, 10]),
#   "sigma": tf.constant([batch_size, 10]),
# }

# Here we parameterize mu and sigma with the output of a linear
# layer. Note that sigma is unconstrained.
params = {}
for name, shape in shapes.items():
  params[name] = linear(x, shape[1])

# Note that you can forward other kwargs to the `Distribution`, like
# `allow_nan_stats` or `name`.
mvn = MultiVariateNormalDiag.from_params(**params, allow_nan_stats=True)
```

Distribution parameters may have constraints (e.g. `sigma` must be positive
for a `Normal` distribution) and the `from_params` method will apply default
parameter transforms. If a user wants to use their own transform, they can
apply it externally and set `make_safe=False`.

##### Args:


*  <b>`make_safe`</b>: Whether the `params` should be constrained. If True,
    `from_params` will apply default parameter transforms. If False, no
    parameter transforms will be applied.
*  <b>`**kwargs`</b>: dict of parameters for the distribution.

##### Returns:

  A distribution parameterized by possibly transformed parameters in
  `kwargs`.

##### Raises:


*  <b>`TypeError`</b>: if `make_safe` is `True` but `_safe_transforms` is not
    implemented directly for `cls`.


- - -

#### `tf.contrib.distributions.Normal.get_batch_shape()` {#Normal.get_batch_shape}

Shape of a single sample from a single event index as a `TensorShape`.

Same meaning as `batch_shape`. May be only partially defined.

##### Returns:


*  <b>`batch_shape`</b>: `TensorShape`, possibly unknown.


- - -

#### `tf.contrib.distributions.Normal.get_event_shape()` {#Normal.get_event_shape}

Shape of a single sample from a single batch as a `TensorShape`.

Same meaning as `event_shape`. May be only partially defined.

##### Returns:


*  <b>`event_shape`</b>: `TensorShape`, possibly unknown.


- - -

#### `tf.contrib.distributions.Normal.is_continuous` {#Normal.is_continuous}




- - -

#### `tf.contrib.distributions.Normal.is_reparameterized` {#Normal.is_reparameterized}




- - -

#### `tf.contrib.distributions.Normal.log_cdf(value, name='log_cdf')` {#Normal.log_cdf}

Log cumulative distribution function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`logcdf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Normal.log_pdf(value, name='log_pdf')` {#Normal.log_pdf}

Log probability density function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if not `is_continuous`.


- - -

#### `tf.contrib.distributions.Normal.log_pmf(value, name='log_pmf')` {#Normal.log_pmf}

Log probability mass function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_pmf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if `is_continuous`.


- - -

#### `tf.contrib.distributions.Normal.log_prob(value, name='log_prob')` {#Normal.log_prob}

Log probability density/mass function (depending on `is_continuous`).

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Normal.mean(name='mean')` {#Normal.mean}

Mean.


- - -

#### `tf.contrib.distributions.Normal.mode(name='mode')` {#Normal.mode}

Mode.


- - -

#### `tf.contrib.distributions.Normal.mu` {#Normal.mu}

Distribution parameter for the mean.


- - -

#### `tf.contrib.distributions.Normal.name` {#Normal.name}

Name prepended to all ops created by this `Distribution`.


- - -

#### `tf.contrib.distributions.Normal.param_shapes(cls, sample_shape, name='DistributionParamShapes')` {#Normal.param_shapes}

Shapes of parameters given the desired shape of a call to `sample()`.

Subclasses should override static method `_param_shapes`.

##### Args:


*  <b>`sample_shape`</b>: `Tensor` or python list/tuple. Desired shape of a call to
    `sample()`.
*  <b>`name`</b>: name to prepend ops with.

##### Returns:

  `dict` of parameter name to `Tensor` shapes.


- - -

#### `tf.contrib.distributions.Normal.param_static_shapes(cls, sample_shape)` {#Normal.param_static_shapes}

param_shapes with static (i.e. TensorShape) shapes.

##### Args:


*  <b>`sample_shape`</b>: `TensorShape` or python list/tuple. Desired shape of a call
    to `sample()`.

##### Returns:

  `dict` of parameter name to `TensorShape`.

##### Raises:


*  <b>`ValueError`</b>: if `sample_shape` is a `TensorShape` and is not fully defined.


- - -

#### `tf.contrib.distributions.Normal.parameters` {#Normal.parameters}

Dictionary of parameters used by this `Distribution`.


- - -

#### `tf.contrib.distributions.Normal.pdf(value, name='pdf')` {#Normal.pdf}

Probability density function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if not `is_continuous`.


- - -

#### `tf.contrib.distributions.Normal.pmf(value, name='pmf')` {#Normal.pmf}

Probability mass function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`pmf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if `is_continuous`.


- - -

#### `tf.contrib.distributions.Normal.prob(value, name='prob')` {#Normal.prob}

Probability density/mass function (depending on `is_continuous`).

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Normal.sample(sample_shape=(), seed=None, name='sample')` {#Normal.sample}

Generate samples of the specified shape.

Note that a call to `sample()` without arguments will generate a single
sample.

##### Args:


*  <b>`sample_shape`</b>: 0D or 1D `int32` `Tensor`. Shape of the generated samples.
*  <b>`seed`</b>: Python integer seed for RNG
*  <b>`name`</b>: name to give to the op.

##### Returns:


*  <b>`samples`</b>: a `Tensor` with prepended dimensions `sample_shape`.


- - -

#### `tf.contrib.distributions.Normal.sample_n(n, seed=None, name='sample_n')` {#Normal.sample_n}

Generate `n` samples.

##### Args:


*  <b>`n`</b>: `Scalar` `Tensor` of type `int32` or `int64`, the number of
    observations to sample.
*  <b>`seed`</b>: Python integer seed for RNG
*  <b>`name`</b>: name to give to the op.

##### Returns:


*  <b>`samples`</b>: a `Tensor` with a prepended dimension (n,).

##### Raises:


*  <b>`TypeError`</b>: if `n` is not an integer type.


- - -

#### `tf.contrib.distributions.Normal.sigma` {#Normal.sigma}

Distribution parameter for standard deviation.


- - -

#### `tf.contrib.distributions.Normal.std(name='std')` {#Normal.std}

Standard deviation.


- - -

#### `tf.contrib.distributions.Normal.validate_args` {#Normal.validate_args}

Python boolean indicated possibly expensive checks are enabled.


- - -

#### `tf.contrib.distributions.Normal.variance(name='variance')` {#Normal.variance}

Variance.



- - -

### `class tf.contrib.distributions.Poisson` {#Poisson}

Poisson distribution.

The Poisson distribution is parameterized by `lam`, the rate parameter.

The pmf of this distribution is:

```

pmf(k) = e^(-lam) * lam^k / k!,  k >= 0
```
- - -

#### `tf.contrib.distributions.Poisson.__init__(lam, validate_args=True, allow_nan_stats=False, name='Poisson')` {#Poisson.__init__}

Construct Poisson distributions.

##### Args:


*  <b>`lam`</b>: Floating point tensor, the rate parameter of the
    distribution(s). `lam` must be positive.
*  <b>`validate_args`</b>: Whether to assert that `lam > 0` as well as inputs to
    pmf computations are non-negative integers. If validate_args is
    `False`, then `pmf` computations might return NaN, as well as
    can be evaluated at any real value.
*  <b>`allow_nan_stats`</b>: Boolean, default `False`.  If `False`, raise an
    exception if a statistic (e.g. mean/mode/etc...) is undefined for any
    batch member.  If `True`, batch members with valid parameters leading to
    undefined statistics will return NaN for this statistic.
*  <b>`name`</b>: A name for this distribution.


- - -

#### `tf.contrib.distributions.Poisson.allow_nan_stats` {#Poisson.allow_nan_stats}

Python boolean describing behavior when a stat is undefined.

Stats return +/- infinity when it makes sense.  E.g., the variance
of a Cauchy distribution is infinity.  However, sometimes the
statistic is undefined, e.g., if a distribution's pdf does not achieve a
maximum within the support of the distribution, the mode is undefined.
If the mean is undefined, then by definition the variance is undefined.
E.g. the mean for Student's T for df = 1 is undefined (no clear way to say
it is either + or - infinity), so the variance = E[(X - mean)^2] is also
undefined.

##### Returns:


*  <b>`allow_nan_stats`</b>: Python boolean.


- - -

#### `tf.contrib.distributions.Poisson.batch_shape(name='batch_shape')` {#Poisson.batch_shape}

Shape of a single sample from a single event index as a 1-D `Tensor`.

The product of the dimensions of the `batch_shape` is the number of
independent distributions of this kind the instance represents.

##### Args:


*  <b>`name`</b>: name to give to the op

##### Returns:


*  <b>`batch_shape`</b>: `Tensor`.


- - -

#### `tf.contrib.distributions.Poisson.cdf(value, name='cdf')` {#Poisson.cdf}

Cumulative distribution function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`cdf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Poisson.dtype` {#Poisson.dtype}

The `DType` of `Tensor`s handled by this `Distribution`.


- - -

#### `tf.contrib.distributions.Poisson.entropy(name='entropy')` {#Poisson.entropy}

Shanon entropy in nats.


- - -

#### `tf.contrib.distributions.Poisson.event_shape(name='event_shape')` {#Poisson.event_shape}

Shape of a single sample from a single batch as a 1-D int32 `Tensor`.

##### Args:


*  <b>`name`</b>: name to give to the op

##### Returns:


*  <b>`event_shape`</b>: `Tensor`.


- - -

#### `tf.contrib.distributions.Poisson.from_params(cls, make_safe=True, **kwargs)` {#Poisson.from_params}

Given (unconstrained) parameters, return an instantiated distribution.

Subclasses should implement a static method `_safe_transforms` that returns
a dict of parameter transforms, which will be used if `make_safe = True`.

Example usage:

```
# Let's say we want a sample of size (batch_size, 10)
shapes = MultiVariateNormalDiag.param_shapes([batch_size, 10])

# shapes has a Tensor shape for mu and sigma
# shapes == {
#   "mu": tf.constant([batch_size, 10]),
#   "sigma": tf.constant([batch_size, 10]),
# }

# Here we parameterize mu and sigma with the output of a linear
# layer. Note that sigma is unconstrained.
params = {}
for name, shape in shapes.items():
  params[name] = linear(x, shape[1])

# Note that you can forward other kwargs to the `Distribution`, like
# `allow_nan_stats` or `name`.
mvn = MultiVariateNormalDiag.from_params(**params, allow_nan_stats=True)
```

Distribution parameters may have constraints (e.g. `sigma` must be positive
for a `Normal` distribution) and the `from_params` method will apply default
parameter transforms. If a user wants to use their own transform, they can
apply it externally and set `make_safe=False`.

##### Args:


*  <b>`make_safe`</b>: Whether the `params` should be constrained. If True,
    `from_params` will apply default parameter transforms. If False, no
    parameter transforms will be applied.
*  <b>`**kwargs`</b>: dict of parameters for the distribution.

##### Returns:

  A distribution parameterized by possibly transformed parameters in
  `kwargs`.

##### Raises:


*  <b>`TypeError`</b>: if `make_safe` is `True` but `_safe_transforms` is not
    implemented directly for `cls`.


- - -

#### `tf.contrib.distributions.Poisson.get_batch_shape()` {#Poisson.get_batch_shape}

Shape of a single sample from a single event index as a `TensorShape`.

Same meaning as `batch_shape`. May be only partially defined.

##### Returns:


*  <b>`batch_shape`</b>: `TensorShape`, possibly unknown.


- - -

#### `tf.contrib.distributions.Poisson.get_event_shape()` {#Poisson.get_event_shape}

Shape of a single sample from a single batch as a `TensorShape`.

Same meaning as `event_shape`. May be only partially defined.

##### Returns:


*  <b>`event_shape`</b>: `TensorShape`, possibly unknown.


- - -

#### `tf.contrib.distributions.Poisson.is_continuous` {#Poisson.is_continuous}




- - -

#### `tf.contrib.distributions.Poisson.is_reparameterized` {#Poisson.is_reparameterized}




- - -

#### `tf.contrib.distributions.Poisson.lam` {#Poisson.lam}

Rate parameter.


- - -

#### `tf.contrib.distributions.Poisson.log_cdf(value, name='log_cdf')` {#Poisson.log_cdf}

Log cumulative distribution function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`logcdf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Poisson.log_pdf(value, name='log_pdf')` {#Poisson.log_pdf}

Log probability density function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if not `is_continuous`.


- - -

#### `tf.contrib.distributions.Poisson.log_pmf(value, name='log_pmf')` {#Poisson.log_pmf}

Log probability mass function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_pmf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if `is_continuous`.


- - -

#### `tf.contrib.distributions.Poisson.log_prob(value, name='log_prob')` {#Poisson.log_prob}

Log probability density/mass function (depending on `is_continuous`).

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Poisson.mean(name='mean')` {#Poisson.mean}

Mean.


- - -

#### `tf.contrib.distributions.Poisson.mode(name='mode')` {#Poisson.mode}

Mode.


- - -

#### `tf.contrib.distributions.Poisson.name` {#Poisson.name}

Name prepended to all ops created by this `Distribution`.


- - -

#### `tf.contrib.distributions.Poisson.param_shapes(cls, sample_shape, name='DistributionParamShapes')` {#Poisson.param_shapes}

Shapes of parameters given the desired shape of a call to `sample()`.

Subclasses should override static method `_param_shapes`.

##### Args:


*  <b>`sample_shape`</b>: `Tensor` or python list/tuple. Desired shape of a call to
    `sample()`.
*  <b>`name`</b>: name to prepend ops with.

##### Returns:

  `dict` of parameter name to `Tensor` shapes.


- - -

#### `tf.contrib.distributions.Poisson.param_static_shapes(cls, sample_shape)` {#Poisson.param_static_shapes}

param_shapes with static (i.e. TensorShape) shapes.

##### Args:


*  <b>`sample_shape`</b>: `TensorShape` or python list/tuple. Desired shape of a call
    to `sample()`.

##### Returns:

  `dict` of parameter name to `TensorShape`.

##### Raises:


*  <b>`ValueError`</b>: if `sample_shape` is a `TensorShape` and is not fully defined.


- - -

#### `tf.contrib.distributions.Poisson.parameters` {#Poisson.parameters}

Dictionary of parameters used by this `Distribution`.


- - -

#### `tf.contrib.distributions.Poisson.pdf(value, name='pdf')` {#Poisson.pdf}

Probability density function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if not `is_continuous`.


- - -

#### `tf.contrib.distributions.Poisson.pmf(value, name='pmf')` {#Poisson.pmf}

Probability mass function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`pmf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if `is_continuous`.


- - -

#### `tf.contrib.distributions.Poisson.prob(value, name='prob')` {#Poisson.prob}

Probability density/mass function (depending on `is_continuous`).

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Poisson.sample(sample_shape=(), seed=None, name='sample')` {#Poisson.sample}

Generate samples of the specified shape.

Note that a call to `sample()` without arguments will generate a single
sample.

##### Args:


*  <b>`sample_shape`</b>: 0D or 1D `int32` `Tensor`. Shape of the generated samples.
*  <b>`seed`</b>: Python integer seed for RNG
*  <b>`name`</b>: name to give to the op.

##### Returns:


*  <b>`samples`</b>: a `Tensor` with prepended dimensions `sample_shape`.


- - -

#### `tf.contrib.distributions.Poisson.sample_n(n, seed=None, name='sample_n')` {#Poisson.sample_n}

Generate `n` samples.

##### Args:


*  <b>`n`</b>: `Scalar` `Tensor` of type `int32` or `int64`, the number of
    observations to sample.
*  <b>`seed`</b>: Python integer seed for RNG
*  <b>`name`</b>: name to give to the op.

##### Returns:


*  <b>`samples`</b>: a `Tensor` with a prepended dimension (n,).

##### Raises:


*  <b>`TypeError`</b>: if `n` is not an integer type.


- - -

#### `tf.contrib.distributions.Poisson.std(name='std')` {#Poisson.std}

Standard deviation.


- - -

#### `tf.contrib.distributions.Poisson.validate_args` {#Poisson.validate_args}

Python boolean indicated possibly expensive checks are enabled.


- - -

#### `tf.contrib.distributions.Poisson.variance(name='variance')` {#Poisson.variance}

Variance.



- - -

### `class tf.contrib.distributions.StudentT` {#StudentT}

Student's t distribution with degree-of-freedom parameter df.

#### Mathematical details

The PDF of this distribution is:

`f(t) = gamma((df+1)/2)/sqrt(df*pi)/gamma(df/2)*(1+t^2/df)^(-(df+1)/2)`

#### Examples

Examples of initialization of one or a batch of distributions.

```python
# Define a single scalar Student t distribution.
single_dist = tf.contrib.distributions.StudentT(df=3)

# Evaluate the pdf at 1, returning a scalar Tensor.
single_dist.pdf(1.)

# Define a batch of two scalar valued Student t's.
# The first has degrees of freedom 2, mean 1, and scale 11.
# The second 3, 2 and 22.
multi_dist = tf.contrib.distributions.StudentT(df=[2, 3],
                                               mu=[1, 2.],
                                               sigma=[11, 22.])

# Evaluate the pdf of the first distribution on 0, and the second on 1.5,
# returning a length two tensor.
multi_dist.pdf([0, 1.5])

# Get 3 samples, returning a 3 x 2 tensor.
multi_dist.sample(3)
```

Arguments are broadcast when possible.

```python
# Define a batch of two Student's t distributions.
# Both have df 2 and mean 1, but different scales.
dist = tf.contrib.distributions.StudentT(df=2, mu=1, sigma=[11, 22.])

# Evaluate the pdf of both distributions on the same point, 3.0,
# returning a length 2 tensor.
dist.pdf(3.0)
```
- - -

#### `tf.contrib.distributions.StudentT.__init__(df, mu, sigma, validate_args=True, allow_nan_stats=False, name='StudentT')` {#StudentT.__init__}

Construct Student's t distributions.

The distributions have degree of freedom `df`, mean `mu`, and scale `sigma`.

The parameters `df`, `mu`, and `sigma` must be shaped in a way that supports
broadcasting (e.g. `df + mu + sigma` is a valid operation).

##### Args:


*  <b>`df`</b>: Floating point tensor, the degrees of freedom of the
    distribution(s). `df` must contain only positive values.
*  <b>`mu`</b>: Floating point tensor, the means of the distribution(s).
*  <b>`sigma`</b>: Floating point tensor, the scaling factor for the
    distribution(s). `sigma` must contain only positive values.
    Note that `sigma` is not the standard deviation of this distribution.
*  <b>`validate_args`</b>: Whether to assert that `df > 0, sigma > 0`. If
    `validate_args` is `False` and inputs are invalid, correct behavior is
    not guaranteed.
*  <b>`allow_nan_stats`</b>: Boolean, default `False`.  If `False`, raise an
    exception if a statistic (e.g. mean/mode/etc...) is undefined for any
    batch member.  If `True`, batch members with valid parameters leading to
    undefined statistics will return NaN for this statistic.
*  <b>`name`</b>: The name to give Ops created by the initializer.

##### Raises:


*  <b>`TypeError`</b>: if mu and sigma are different dtypes.


- - -

#### `tf.contrib.distributions.StudentT.allow_nan_stats` {#StudentT.allow_nan_stats}

Python boolean describing behavior when a stat is undefined.

Stats return +/- infinity when it makes sense.  E.g., the variance
of a Cauchy distribution is infinity.  However, sometimes the
statistic is undefined, e.g., if a distribution's pdf does not achieve a
maximum within the support of the distribution, the mode is undefined.
If the mean is undefined, then by definition the variance is undefined.
E.g. the mean for Student's T for df = 1 is undefined (no clear way to say
it is either + or - infinity), so the variance = E[(X - mean)^2] is also
undefined.

##### Returns:


*  <b>`allow_nan_stats`</b>: Python boolean.


- - -

#### `tf.contrib.distributions.StudentT.batch_shape(name='batch_shape')` {#StudentT.batch_shape}

Shape of a single sample from a single event index as a 1-D `Tensor`.

The product of the dimensions of the `batch_shape` is the number of
independent distributions of this kind the instance represents.

##### Args:


*  <b>`name`</b>: name to give to the op

##### Returns:


*  <b>`batch_shape`</b>: `Tensor`.


- - -

#### `tf.contrib.distributions.StudentT.cdf(value, name='cdf')` {#StudentT.cdf}

Cumulative distribution function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`cdf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.StudentT.df` {#StudentT.df}

Degrees of freedom in these Student's t distribution(s).


- - -

#### `tf.contrib.distributions.StudentT.dtype` {#StudentT.dtype}

The `DType` of `Tensor`s handled by this `Distribution`.


- - -

#### `tf.contrib.distributions.StudentT.entropy(name='entropy')` {#StudentT.entropy}

Shanon entropy in nats.


- - -

#### `tf.contrib.distributions.StudentT.event_shape(name='event_shape')` {#StudentT.event_shape}

Shape of a single sample from a single batch as a 1-D int32 `Tensor`.

##### Args:


*  <b>`name`</b>: name to give to the op

##### Returns:


*  <b>`event_shape`</b>: `Tensor`.


- - -

#### `tf.contrib.distributions.StudentT.from_params(cls, make_safe=True, **kwargs)` {#StudentT.from_params}

Given (unconstrained) parameters, return an instantiated distribution.

Subclasses should implement a static method `_safe_transforms` that returns
a dict of parameter transforms, which will be used if `make_safe = True`.

Example usage:

```
# Let's say we want a sample of size (batch_size, 10)
shapes = MultiVariateNormalDiag.param_shapes([batch_size, 10])

# shapes has a Tensor shape for mu and sigma
# shapes == {
#   "mu": tf.constant([batch_size, 10]),
#   "sigma": tf.constant([batch_size, 10]),
# }

# Here we parameterize mu and sigma with the output of a linear
# layer. Note that sigma is unconstrained.
params = {}
for name, shape in shapes.items():
  params[name] = linear(x, shape[1])

# Note that you can forward other kwargs to the `Distribution`, like
# `allow_nan_stats` or `name`.
mvn = MultiVariateNormalDiag.from_params(**params, allow_nan_stats=True)
```

Distribution parameters may have constraints (e.g. `sigma` must be positive
for a `Normal` distribution) and the `from_params` method will apply default
parameter transforms. If a user wants to use their own transform, they can
apply it externally and set `make_safe=False`.

##### Args:


*  <b>`make_safe`</b>: Whether the `params` should be constrained. If True,
    `from_params` will apply default parameter transforms. If False, no
    parameter transforms will be applied.
*  <b>`**kwargs`</b>: dict of parameters for the distribution.

##### Returns:

  A distribution parameterized by possibly transformed parameters in
  `kwargs`.

##### Raises:


*  <b>`TypeError`</b>: if `make_safe` is `True` but `_safe_transforms` is not
    implemented directly for `cls`.


- - -

#### `tf.contrib.distributions.StudentT.get_batch_shape()` {#StudentT.get_batch_shape}

Shape of a single sample from a single event index as a `TensorShape`.

Same meaning as `batch_shape`. May be only partially defined.

##### Returns:


*  <b>`batch_shape`</b>: `TensorShape`, possibly unknown.


- - -

#### `tf.contrib.distributions.StudentT.get_event_shape()` {#StudentT.get_event_shape}

Shape of a single sample from a single batch as a `TensorShape`.

Same meaning as `event_shape`. May be only partially defined.

##### Returns:


*  <b>`event_shape`</b>: `TensorShape`, possibly unknown.


- - -

#### `tf.contrib.distributions.StudentT.is_continuous` {#StudentT.is_continuous}




- - -

#### `tf.contrib.distributions.StudentT.is_reparameterized` {#StudentT.is_reparameterized}




- - -

#### `tf.contrib.distributions.StudentT.log_cdf(value, name='log_cdf')` {#StudentT.log_cdf}

Log cumulative distribution function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`logcdf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.StudentT.log_pdf(value, name='log_pdf')` {#StudentT.log_pdf}

Log probability density function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if not `is_continuous`.


- - -

#### `tf.contrib.distributions.StudentT.log_pmf(value, name='log_pmf')` {#StudentT.log_pmf}

Log probability mass function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_pmf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if `is_continuous`.


- - -

#### `tf.contrib.distributions.StudentT.log_prob(value, name='log_prob')` {#StudentT.log_prob}

Log probability density/mass function (depending on `is_continuous`).

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.StudentT.mean(name='mean')` {#StudentT.mean}

Mean.


- - -

#### `tf.contrib.distributions.StudentT.mode(name='mode')` {#StudentT.mode}

Mode.


- - -

#### `tf.contrib.distributions.StudentT.mu` {#StudentT.mu}

Locations of these Student's t distribution(s).


- - -

#### `tf.contrib.distributions.StudentT.name` {#StudentT.name}

Name prepended to all ops created by this `Distribution`.


- - -

#### `tf.contrib.distributions.StudentT.param_shapes(cls, sample_shape, name='DistributionParamShapes')` {#StudentT.param_shapes}

Shapes of parameters given the desired shape of a call to `sample()`.

Subclasses should override static method `_param_shapes`.

##### Args:


*  <b>`sample_shape`</b>: `Tensor` or python list/tuple. Desired shape of a call to
    `sample()`.
*  <b>`name`</b>: name to prepend ops with.

##### Returns:

  `dict` of parameter name to `Tensor` shapes.


- - -

#### `tf.contrib.distributions.StudentT.param_static_shapes(cls, sample_shape)` {#StudentT.param_static_shapes}

param_shapes with static (i.e. TensorShape) shapes.

##### Args:


*  <b>`sample_shape`</b>: `TensorShape` or python list/tuple. Desired shape of a call
    to `sample()`.

##### Returns:

  `dict` of parameter name to `TensorShape`.

##### Raises:


*  <b>`ValueError`</b>: if `sample_shape` is a `TensorShape` and is not fully defined.


- - -

#### `tf.contrib.distributions.StudentT.parameters` {#StudentT.parameters}

Dictionary of parameters used by this `Distribution`.


- - -

#### `tf.contrib.distributions.StudentT.pdf(value, name='pdf')` {#StudentT.pdf}

Probability density function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if not `is_continuous`.


- - -

#### `tf.contrib.distributions.StudentT.pmf(value, name='pmf')` {#StudentT.pmf}

Probability mass function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`pmf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if `is_continuous`.


- - -

#### `tf.contrib.distributions.StudentT.prob(value, name='prob')` {#StudentT.prob}

Probability density/mass function (depending on `is_continuous`).

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.StudentT.sample(sample_shape=(), seed=None, name='sample')` {#StudentT.sample}

Generate samples of the specified shape.

Note that a call to `sample()` without arguments will generate a single
sample.

##### Args:


*  <b>`sample_shape`</b>: 0D or 1D `int32` `Tensor`. Shape of the generated samples.
*  <b>`seed`</b>: Python integer seed for RNG
*  <b>`name`</b>: name to give to the op.

##### Returns:


*  <b>`samples`</b>: a `Tensor` with prepended dimensions `sample_shape`.


- - -

#### `tf.contrib.distributions.StudentT.sample_n(n, seed=None, name='sample_n')` {#StudentT.sample_n}

Generate `n` samples.

##### Args:


*  <b>`n`</b>: `Scalar` `Tensor` of type `int32` or `int64`, the number of
    observations to sample.
*  <b>`seed`</b>: Python integer seed for RNG
*  <b>`name`</b>: name to give to the op.

##### Returns:


*  <b>`samples`</b>: a `Tensor` with a prepended dimension (n,).

##### Raises:


*  <b>`TypeError`</b>: if `n` is not an integer type.


- - -

#### `tf.contrib.distributions.StudentT.sigma` {#StudentT.sigma}

Scaling factors of these Student's t distribution(s).


- - -

#### `tf.contrib.distributions.StudentT.std(name='std')` {#StudentT.std}

Standard deviation.


- - -

#### `tf.contrib.distributions.StudentT.validate_args` {#StudentT.validate_args}

Python boolean indicated possibly expensive checks are enabled.


- - -

#### `tf.contrib.distributions.StudentT.variance(name='variance')` {#StudentT.variance}

Variance.



- - -

### `class tf.contrib.distributions.Uniform` {#Uniform}

Uniform distribution with `a` and `b` parameters.

The PDF of this distribution is constant between [`a`, `b`], and 0 elsewhere.
- - -

#### `tf.contrib.distributions.Uniform.__init__(a=0.0, b=1.0, validate_args=True, allow_nan_stats=False, name='Uniform')` {#Uniform.__init__}

Construct Uniform distributions with `a` and `b`.

The parameters `a` and `b` must be shaped in a way that supports
broadcasting (e.g. `b - a` is a valid operation).

Here are examples without broadcasting:

```python
# Without broadcasting
u1 = Uniform(3.0, 4.0)  # a single uniform distribution [3, 4]
u2 = Uniform([1.0, 2.0], [3.0, 4.0])  # 2 distributions [1, 3], [2, 4]
u3 = Uniform([[1.0, 2.0],
              [3.0, 4.0]],
             [[1.5, 2.5],
              [3.5, 4.5]])  # 4 distributions
```

And with broadcasting:

```python
u1 = Uniform(3.0, [5.0, 6.0, 7.0])  # 3 distributions
```

##### Args:


*  <b>`a`</b>: Floating point tensor, the minimum endpoint.
*  <b>`b`</b>: Floating point tensor, the maximum endpoint. Must be > `a`.
*  <b>`validate_args`</b>: Whether to assert that `a > b`. If `validate_args` is
    `False` and inputs are invalid, correct behavior is not guaranteed.
*  <b>`allow_nan_stats`</b>: Boolean, default `False`.  If `False`, raise an
    exception if a statistic (e.g. mean/mode/etc...) is undefined for any
    batch member.  If `True`, batch members with valid parameters leading to
    undefined statistics will return NaN for this statistic.
*  <b>`name`</b>: The name to prefix Ops created by this distribution class.

##### Raises:


*  <b>`InvalidArgumentError`</b>: if `a >= b` and `validate_args=True`.


- - -

#### `tf.contrib.distributions.Uniform.a` {#Uniform.a}




- - -

#### `tf.contrib.distributions.Uniform.allow_nan_stats` {#Uniform.allow_nan_stats}

Python boolean describing behavior when a stat is undefined.

Stats return +/- infinity when it makes sense.  E.g., the variance
of a Cauchy distribution is infinity.  However, sometimes the
statistic is undefined, e.g., if a distribution's pdf does not achieve a
maximum within the support of the distribution, the mode is undefined.
If the mean is undefined, then by definition the variance is undefined.
E.g. the mean for Student's T for df = 1 is undefined (no clear way to say
it is either + or - infinity), so the variance = E[(X - mean)^2] is also
undefined.

##### Returns:


*  <b>`allow_nan_stats`</b>: Python boolean.


- - -

#### `tf.contrib.distributions.Uniform.b` {#Uniform.b}




- - -

#### `tf.contrib.distributions.Uniform.batch_shape(name='batch_shape')` {#Uniform.batch_shape}

Shape of a single sample from a single event index as a 1-D `Tensor`.

The product of the dimensions of the `batch_shape` is the number of
independent distributions of this kind the instance represents.

##### Args:


*  <b>`name`</b>: name to give to the op

##### Returns:


*  <b>`batch_shape`</b>: `Tensor`.


- - -

#### `tf.contrib.distributions.Uniform.cdf(value, name='cdf')` {#Uniform.cdf}

Cumulative distribution function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`cdf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Uniform.dtype` {#Uniform.dtype}

The `DType` of `Tensor`s handled by this `Distribution`.


- - -

#### `tf.contrib.distributions.Uniform.entropy(name='entropy')` {#Uniform.entropy}

Shanon entropy in nats.


- - -

#### `tf.contrib.distributions.Uniform.event_shape(name='event_shape')` {#Uniform.event_shape}

Shape of a single sample from a single batch as a 1-D int32 `Tensor`.

##### Args:


*  <b>`name`</b>: name to give to the op

##### Returns:


*  <b>`event_shape`</b>: `Tensor`.


- - -

#### `tf.contrib.distributions.Uniform.from_params(cls, make_safe=True, **kwargs)` {#Uniform.from_params}

Given (unconstrained) parameters, return an instantiated distribution.

Subclasses should implement a static method `_safe_transforms` that returns
a dict of parameter transforms, which will be used if `make_safe = True`.

Example usage:

```
# Let's say we want a sample of size (batch_size, 10)
shapes = MultiVariateNormalDiag.param_shapes([batch_size, 10])

# shapes has a Tensor shape for mu and sigma
# shapes == {
#   "mu": tf.constant([batch_size, 10]),
#   "sigma": tf.constant([batch_size, 10]),
# }

# Here we parameterize mu and sigma with the output of a linear
# layer. Note that sigma is unconstrained.
params = {}
for name, shape in shapes.items():
  params[name] = linear(x, shape[1])

# Note that you can forward other kwargs to the `Distribution`, like
# `allow_nan_stats` or `name`.
mvn = MultiVariateNormalDiag.from_params(**params, allow_nan_stats=True)
```

Distribution parameters may have constraints (e.g. `sigma` must be positive
for a `Normal` distribution) and the `from_params` method will apply default
parameter transforms. If a user wants to use their own transform, they can
apply it externally and set `make_safe=False`.

##### Args:


*  <b>`make_safe`</b>: Whether the `params` should be constrained. If True,
    `from_params` will apply default parameter transforms. If False, no
    parameter transforms will be applied.
*  <b>`**kwargs`</b>: dict of parameters for the distribution.

##### Returns:

  A distribution parameterized by possibly transformed parameters in
  `kwargs`.

##### Raises:


*  <b>`TypeError`</b>: if `make_safe` is `True` but `_safe_transforms` is not
    implemented directly for `cls`.


- - -

#### `tf.contrib.distributions.Uniform.get_batch_shape()` {#Uniform.get_batch_shape}

Shape of a single sample from a single event index as a `TensorShape`.

Same meaning as `batch_shape`. May be only partially defined.

##### Returns:


*  <b>`batch_shape`</b>: `TensorShape`, possibly unknown.


- - -

#### `tf.contrib.distributions.Uniform.get_event_shape()` {#Uniform.get_event_shape}

Shape of a single sample from a single batch as a `TensorShape`.

Same meaning as `event_shape`. May be only partially defined.

##### Returns:


*  <b>`event_shape`</b>: `TensorShape`, possibly unknown.


- - -

#### `tf.contrib.distributions.Uniform.is_continuous` {#Uniform.is_continuous}




- - -

#### `tf.contrib.distributions.Uniform.is_reparameterized` {#Uniform.is_reparameterized}




- - -

#### `tf.contrib.distributions.Uniform.log_cdf(value, name='log_cdf')` {#Uniform.log_cdf}

Log cumulative distribution function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`logcdf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Uniform.log_pdf(value, name='log_pdf')` {#Uniform.log_pdf}

Log probability density function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if not `is_continuous`.


- - -

#### `tf.contrib.distributions.Uniform.log_pmf(value, name='log_pmf')` {#Uniform.log_pmf}

Log probability mass function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_pmf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if `is_continuous`.


- - -

#### `tf.contrib.distributions.Uniform.log_prob(value, name='log_prob')` {#Uniform.log_prob}

Log probability density/mass function (depending on `is_continuous`).

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Uniform.mean(name='mean')` {#Uniform.mean}

Mean.


- - -

#### `tf.contrib.distributions.Uniform.mode(name='mode')` {#Uniform.mode}

Mode.


- - -

#### `tf.contrib.distributions.Uniform.name` {#Uniform.name}

Name prepended to all ops created by this `Distribution`.


- - -

#### `tf.contrib.distributions.Uniform.param_shapes(cls, sample_shape, name='DistributionParamShapes')` {#Uniform.param_shapes}

Shapes of parameters given the desired shape of a call to `sample()`.

Subclasses should override static method `_param_shapes`.

##### Args:


*  <b>`sample_shape`</b>: `Tensor` or python list/tuple. Desired shape of a call to
    `sample()`.
*  <b>`name`</b>: name to prepend ops with.

##### Returns:

  `dict` of parameter name to `Tensor` shapes.


- - -

#### `tf.contrib.distributions.Uniform.param_static_shapes(cls, sample_shape)` {#Uniform.param_static_shapes}

param_shapes with static (i.e. TensorShape) shapes.

##### Args:


*  <b>`sample_shape`</b>: `TensorShape` or python list/tuple. Desired shape of a call
    to `sample()`.

##### Returns:

  `dict` of parameter name to `TensorShape`.

##### Raises:


*  <b>`ValueError`</b>: if `sample_shape` is a `TensorShape` and is not fully defined.


- - -

#### `tf.contrib.distributions.Uniform.parameters` {#Uniform.parameters}

Dictionary of parameters used by this `Distribution`.


- - -

#### `tf.contrib.distributions.Uniform.pdf(value, name='pdf')` {#Uniform.pdf}

Probability density function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if not `is_continuous`.


- - -

#### `tf.contrib.distributions.Uniform.pmf(value, name='pmf')` {#Uniform.pmf}

Probability mass function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`pmf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if `is_continuous`.


- - -

#### `tf.contrib.distributions.Uniform.prob(value, name='prob')` {#Uniform.prob}

Probability density/mass function (depending on `is_continuous`).

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Uniform.range(name='range')` {#Uniform.range}

`b - a`.


- - -

#### `tf.contrib.distributions.Uniform.sample(sample_shape=(), seed=None, name='sample')` {#Uniform.sample}

Generate samples of the specified shape.

Note that a call to `sample()` without arguments will generate a single
sample.

##### Args:


*  <b>`sample_shape`</b>: 0D or 1D `int32` `Tensor`. Shape of the generated samples.
*  <b>`seed`</b>: Python integer seed for RNG
*  <b>`name`</b>: name to give to the op.

##### Returns:


*  <b>`samples`</b>: a `Tensor` with prepended dimensions `sample_shape`.


- - -

#### `tf.contrib.distributions.Uniform.sample_n(n, seed=None, name='sample_n')` {#Uniform.sample_n}

Generate `n` samples.

##### Args:


*  <b>`n`</b>: `Scalar` `Tensor` of type `int32` or `int64`, the number of
    observations to sample.
*  <b>`seed`</b>: Python integer seed for RNG
*  <b>`name`</b>: name to give to the op.

##### Returns:


*  <b>`samples`</b>: a `Tensor` with a prepended dimension (n,).

##### Raises:


*  <b>`TypeError`</b>: if `n` is not an integer type.


- - -

#### `tf.contrib.distributions.Uniform.std(name='std')` {#Uniform.std}

Standard deviation.


- - -

#### `tf.contrib.distributions.Uniform.validate_args` {#Uniform.validate_args}

Python boolean indicated possibly expensive checks are enabled.


- - -

#### `tf.contrib.distributions.Uniform.variance(name='variance')` {#Uniform.variance}

Variance.




### Multivariate distributions

#### Multivariate normal

- - -

### `class tf.contrib.distributions.MultivariateNormalDiag` {#MultivariateNormalDiag}

The multivariate normal distribution on `R^k`.

This distribution is defined by a 1-D mean `mu` and a 1-D diagonal
`diag_stdev`, representing the standard deviations.  This distribution
assumes the random variables, `(X_1,...,X_k)` are independent, thus no
non-diagonal terms of the covariance matrix are needed.

This allows for `O(k)` pdf evaluation, sampling, and storage.

#### Mathematical details

The PDF of this distribution is defined in terms of the diagonal covariance
determined by `diag_stdev`: `C_{ii} = diag_stdev[i]**2`.

```
f(x) = (2 pi)^(-k/2) |det(C)|^(-1/2) exp(-1/2 (x - mu)^T C^{-1} (x - mu))
```

#### Examples

A single multi-variate Gaussian distribution is defined by a vector of means
of length `k`, and the square roots of the (independent) random variables.

Extra leading dimensions, if provided, allow for batches.

```python
# Initialize a single 3-variate Gaussian with diagonal standard deviation.
mu = [1, 2, 3.]
diag_stdev = [4, 5, 6.]
dist = tf.contrib.distributions.MultivariateNormalDiag(mu, diag_stdev)

# Evaluate this on an observation in R^3, returning a scalar.
dist.pdf([-1, 0, 1])

# Initialize a batch of two 3-variate Gaussians.
mu = [[1, 2, 3], [11, 22, 33]]  # shape 2 x 3
diag_stdev = ...  # shape 2 x 3, positive.
dist = tf.contrib.distributions.MultivariateNormalDiag(mu, diag_stdev)

# Evaluate this on a two observations, each in R^3, returning a length two
# tensor.
x = [[-1, 0, 1], [-11, 0, 11]]  # Shape 2 x 3.
dist.pdf(x)
```
- - -

#### `tf.contrib.distributions.MultivariateNormalDiag.__init__(mu, diag_stdev, validate_args=True, allow_nan_stats=False, name='MultivariateNormalDiag')` {#MultivariateNormalDiag.__init__}

Multivariate Normal distributions on `R^k`.

User must provide means `mu` and standard deviations `diag_stdev`.
Each batch member represents a random vector `(X_1,...,X_k)` of independent
random normals.
The mean of `X_i` is `mu[i]`, and the standard deviation is `diag_stdev[i]`.

##### Args:


*  <b>`mu`</b>: Rank `N + 1` floating point tensor with shape `[N1,...,Nb, k]`,
    `b >= 0`.
*  <b>`diag_stdev`</b>: Rank `N + 1` `Tensor` with same `dtype` and shape as `mu`,
    representing the standard deviations.  Must be positive.
*  <b>`validate_args`</b>: Whether to validate input with asserts.  If `validate_args`
    is `False`,
    and the inputs are invalid, correct behavior is not guaranteed.
*  <b>`allow_nan_stats`</b>: `Boolean`, default `False`.  If `False`, raise an
    exception if a statistic (e.g. mean/mode/etc...) is undefined for any
    batch member If `True`, batch members with valid parameters leading to
    undefined statistics will return NaN for this statistic.
*  <b>`name`</b>: The name to give Ops created by the initializer.

##### Raises:


*  <b>`TypeError`</b>: If `mu` and `diag_stdev` are different dtypes.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiag.allow_nan_stats` {#MultivariateNormalDiag.allow_nan_stats}

Python boolean describing behavior when a stat is undefined.

Stats return +/- infinity when it makes sense.  E.g., the variance
of a Cauchy distribution is infinity.  However, sometimes the
statistic is undefined, e.g., if a distribution's pdf does not achieve a
maximum within the support of the distribution, the mode is undefined.
If the mean is undefined, then by definition the variance is undefined.
E.g. the mean for Student's T for df = 1 is undefined (no clear way to say
it is either + or - infinity), so the variance = E[(X - mean)^2] is also
undefined.

##### Returns:


*  <b>`allow_nan_stats`</b>: Python boolean.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiag.batch_shape(name='batch_shape')` {#MultivariateNormalDiag.batch_shape}

Shape of a single sample from a single event index as a 1-D `Tensor`.

The product of the dimensions of the `batch_shape` is the number of
independent distributions of this kind the instance represents.

##### Args:


*  <b>`name`</b>: name to give to the op

##### Returns:


*  <b>`batch_shape`</b>: `Tensor`.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiag.cdf(value, name='cdf')` {#MultivariateNormalDiag.cdf}

Cumulative distribution function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`cdf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiag.dtype` {#MultivariateNormalDiag.dtype}

The `DType` of `Tensor`s handled by this `Distribution`.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiag.entropy(name='entropy')` {#MultivariateNormalDiag.entropy}

Shanon entropy in nats.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiag.event_shape(name='event_shape')` {#MultivariateNormalDiag.event_shape}

Shape of a single sample from a single batch as a 1-D int32 `Tensor`.

##### Args:


*  <b>`name`</b>: name to give to the op

##### Returns:


*  <b>`event_shape`</b>: `Tensor`.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiag.from_params(cls, make_safe=True, **kwargs)` {#MultivariateNormalDiag.from_params}

Given (unconstrained) parameters, return an instantiated distribution.

Subclasses should implement a static method `_safe_transforms` that returns
a dict of parameter transforms, which will be used if `make_safe = True`.

Example usage:

```
# Let's say we want a sample of size (batch_size, 10)
shapes = MultiVariateNormalDiag.param_shapes([batch_size, 10])

# shapes has a Tensor shape for mu and sigma
# shapes == {
#   "mu": tf.constant([batch_size, 10]),
#   "sigma": tf.constant([batch_size, 10]),
# }

# Here we parameterize mu and sigma with the output of a linear
# layer. Note that sigma is unconstrained.
params = {}
for name, shape in shapes.items():
  params[name] = linear(x, shape[1])

# Note that you can forward other kwargs to the `Distribution`, like
# `allow_nan_stats` or `name`.
mvn = MultiVariateNormalDiag.from_params(**params, allow_nan_stats=True)
```

Distribution parameters may have constraints (e.g. `sigma` must be positive
for a `Normal` distribution) and the `from_params` method will apply default
parameter transforms. If a user wants to use their own transform, they can
apply it externally and set `make_safe=False`.

##### Args:


*  <b>`make_safe`</b>: Whether the `params` should be constrained. If True,
    `from_params` will apply default parameter transforms. If False, no
    parameter transforms will be applied.
*  <b>`**kwargs`</b>: dict of parameters for the distribution.

##### Returns:

  A distribution parameterized by possibly transformed parameters in
  `kwargs`.

##### Raises:


*  <b>`TypeError`</b>: if `make_safe` is `True` but `_safe_transforms` is not
    implemented directly for `cls`.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiag.get_batch_shape()` {#MultivariateNormalDiag.get_batch_shape}

Shape of a single sample from a single event index as a `TensorShape`.

Same meaning as `batch_shape`. May be only partially defined.

##### Returns:


*  <b>`batch_shape`</b>: `TensorShape`, possibly unknown.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiag.get_event_shape()` {#MultivariateNormalDiag.get_event_shape}

Shape of a single sample from a single batch as a `TensorShape`.

Same meaning as `event_shape`. May be only partially defined.

##### Returns:


*  <b>`event_shape`</b>: `TensorShape`, possibly unknown.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiag.is_continuous` {#MultivariateNormalDiag.is_continuous}




- - -

#### `tf.contrib.distributions.MultivariateNormalDiag.is_reparameterized` {#MultivariateNormalDiag.is_reparameterized}




- - -

#### `tf.contrib.distributions.MultivariateNormalDiag.log_cdf(value, name='log_cdf')` {#MultivariateNormalDiag.log_cdf}

Log cumulative distribution function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`logcdf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiag.log_pdf(value, name='log_pdf')` {#MultivariateNormalDiag.log_pdf}

Log probability density function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if not `is_continuous`.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiag.log_pmf(value, name='log_pmf')` {#MultivariateNormalDiag.log_pmf}

Log probability mass function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_pmf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if `is_continuous`.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiag.log_prob(value, name='log_prob')` {#MultivariateNormalDiag.log_prob}

Log probability density/mass function (depending on `is_continuous`).

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiag.log_sigma_det(name='log_sigma_det')` {#MultivariateNormalDiag.log_sigma_det}

Log of determinant of covariance matrix.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiag.mean(name='mean')` {#MultivariateNormalDiag.mean}

Mean.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiag.mode(name='mode')` {#MultivariateNormalDiag.mode}

Mode.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiag.mu` {#MultivariateNormalDiag.mu}




- - -

#### `tf.contrib.distributions.MultivariateNormalDiag.name` {#MultivariateNormalDiag.name}

Name prepended to all ops created by this `Distribution`.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiag.param_shapes(cls, sample_shape, name='DistributionParamShapes')` {#MultivariateNormalDiag.param_shapes}

Shapes of parameters given the desired shape of a call to `sample()`.

Subclasses should override static method `_param_shapes`.

##### Args:


*  <b>`sample_shape`</b>: `Tensor` or python list/tuple. Desired shape of a call to
    `sample()`.
*  <b>`name`</b>: name to prepend ops with.

##### Returns:

  `dict` of parameter name to `Tensor` shapes.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiag.param_static_shapes(cls, sample_shape)` {#MultivariateNormalDiag.param_static_shapes}

param_shapes with static (i.e. TensorShape) shapes.

##### Args:


*  <b>`sample_shape`</b>: `TensorShape` or python list/tuple. Desired shape of a call
    to `sample()`.

##### Returns:

  `dict` of parameter name to `TensorShape`.

##### Raises:


*  <b>`ValueError`</b>: if `sample_shape` is a `TensorShape` and is not fully defined.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiag.parameters` {#MultivariateNormalDiag.parameters}

Dictionary of parameters used by this `Distribution`.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiag.pdf(value, name='pdf')` {#MultivariateNormalDiag.pdf}

Probability density function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if not `is_continuous`.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiag.pmf(value, name='pmf')` {#MultivariateNormalDiag.pmf}

Probability mass function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`pmf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if `is_continuous`.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiag.prob(value, name='prob')` {#MultivariateNormalDiag.prob}

Probability density/mass function (depending on `is_continuous`).

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiag.sample(sample_shape=(), seed=None, name='sample')` {#MultivariateNormalDiag.sample}

Generate samples of the specified shape.

Note that a call to `sample()` without arguments will generate a single
sample.

##### Args:


*  <b>`sample_shape`</b>: 0D or 1D `int32` `Tensor`. Shape of the generated samples.
*  <b>`seed`</b>: Python integer seed for RNG
*  <b>`name`</b>: name to give to the op.

##### Returns:


*  <b>`samples`</b>: a `Tensor` with prepended dimensions `sample_shape`.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiag.sample_n(n, seed=None, name='sample_n')` {#MultivariateNormalDiag.sample_n}

Generate `n` samples.

##### Args:


*  <b>`n`</b>: `Scalar` `Tensor` of type `int32` or `int64`, the number of
    observations to sample.
*  <b>`seed`</b>: Python integer seed for RNG
*  <b>`name`</b>: name to give to the op.

##### Returns:


*  <b>`samples`</b>: a `Tensor` with a prepended dimension (n,).

##### Raises:


*  <b>`TypeError`</b>: if `n` is not an integer type.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiag.sigma` {#MultivariateNormalDiag.sigma}

Dense (batch) covariance matrix, if available.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiag.sigma_det(name='sigma_det')` {#MultivariateNormalDiag.sigma_det}

Determinant of covariance matrix.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiag.std(name='std')` {#MultivariateNormalDiag.std}

Standard deviation.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiag.validate_args` {#MultivariateNormalDiag.validate_args}

Python boolean indicated possibly expensive checks are enabled.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiag.variance(name='variance')` {#MultivariateNormalDiag.variance}

Variance.



- - -

### `class tf.contrib.distributions.MultivariateNormalFull` {#MultivariateNormalFull}

The multivariate normal distribution on `R^k`.

This distribution is defined by a 1-D mean `mu` and covariance matrix `sigma`.
Evaluation of the pdf, determinant, and sampling are all `O(k^3)` operations.

#### Mathematical details

With `C = sigma`, the PDF of this distribution is:

```
f(x) = (2 pi)^(-k/2) |det(C)|^(-1/2) exp(-1/2 (x - mu)^T C^{-1} (x - mu))
```

#### Examples

A single multi-variate Gaussian distribution is defined by a vector of means
of length `k`, and a covariance matrix of shape `k x k`.

Extra leading dimensions, if provided, allow for batches.

```python
# Initialize a single 3-variate Gaussian with diagonal covariance.
mu = [1, 2, 3.]
sigma = [[1, 0, 0], [0, 3, 0], [0, 0, 2.]]
dist = tf.contrib.distributions.MultivariateNormalFull(mu, chol)

# Evaluate this on an observation in R^3, returning a scalar.
dist.pdf([-1, 0, 1])

# Initialize a batch of two 3-variate Gaussians.
mu = [[1, 2, 3], [11, 22, 33.]]
sigma = ...  # shape 2 x 3 x 3, positive definite.
dist = tf.contrib.distributions.MultivariateNormalFull(mu, sigma)

# Evaluate this on a two observations, each in R^3, returning a length two
# tensor.
x = [[-1, 0, 1], [-11, 0, 11.]]  # Shape 2 x 3.
dist.pdf(x)
```
- - -

#### `tf.contrib.distributions.MultivariateNormalFull.__init__(mu, sigma, validate_args=True, allow_nan_stats=False, name='MultivariateNormalFull')` {#MultivariateNormalFull.__init__}

Multivariate Normal distributions on `R^k`.

User must provide means `mu` and `sigma`, the mean and covariance.

##### Args:


*  <b>`mu`</b>: `(N+1)-D` floating point tensor with shape `[N1,...,Nb, k]`,
    `b >= 0`.
*  <b>`sigma`</b>: `(N+2)-D` `Tensor` with same `dtype` as `mu` and shape
    `[N1,...,Nb, k, k]`.  Each batch member must be positive definite.
*  <b>`validate_args`</b>: Whether to validate input with asserts.  If `validate_args`
    is `False`, and the inputs are invalid, correct behavior is not
    guaranteed.
*  <b>`allow_nan_stats`</b>: `Boolean`, default `False`.  If `False`, raise an
    exception if a statistic (e.g. mean/mode/etc...) is undefined for any
    batch member If `True`, batch members with valid parameters leading to
    undefined statistics will return NaN for this statistic.
*  <b>`name`</b>: The name to give Ops created by the initializer.

##### Raises:


*  <b>`TypeError`</b>: If `mu` and `sigma` are different dtypes.


- - -

#### `tf.contrib.distributions.MultivariateNormalFull.allow_nan_stats` {#MultivariateNormalFull.allow_nan_stats}

Python boolean describing behavior when a stat is undefined.

Stats return +/- infinity when it makes sense.  E.g., the variance
of a Cauchy distribution is infinity.  However, sometimes the
statistic is undefined, e.g., if a distribution's pdf does not achieve a
maximum within the support of the distribution, the mode is undefined.
If the mean is undefined, then by definition the variance is undefined.
E.g. the mean for Student's T for df = 1 is undefined (no clear way to say
it is either + or - infinity), so the variance = E[(X - mean)^2] is also
undefined.

##### Returns:


*  <b>`allow_nan_stats`</b>: Python boolean.


- - -

#### `tf.contrib.distributions.MultivariateNormalFull.batch_shape(name='batch_shape')` {#MultivariateNormalFull.batch_shape}

Shape of a single sample from a single event index as a 1-D `Tensor`.

The product of the dimensions of the `batch_shape` is the number of
independent distributions of this kind the instance represents.

##### Args:


*  <b>`name`</b>: name to give to the op

##### Returns:


*  <b>`batch_shape`</b>: `Tensor`.


- - -

#### `tf.contrib.distributions.MultivariateNormalFull.cdf(value, name='cdf')` {#MultivariateNormalFull.cdf}

Cumulative distribution function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`cdf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.MultivariateNormalFull.dtype` {#MultivariateNormalFull.dtype}

The `DType` of `Tensor`s handled by this `Distribution`.


- - -

#### `tf.contrib.distributions.MultivariateNormalFull.entropy(name='entropy')` {#MultivariateNormalFull.entropy}

Shanon entropy in nats.


- - -

#### `tf.contrib.distributions.MultivariateNormalFull.event_shape(name='event_shape')` {#MultivariateNormalFull.event_shape}

Shape of a single sample from a single batch as a 1-D int32 `Tensor`.

##### Args:


*  <b>`name`</b>: name to give to the op

##### Returns:


*  <b>`event_shape`</b>: `Tensor`.


- - -

#### `tf.contrib.distributions.MultivariateNormalFull.from_params(cls, make_safe=True, **kwargs)` {#MultivariateNormalFull.from_params}

Given (unconstrained) parameters, return an instantiated distribution.

Subclasses should implement a static method `_safe_transforms` that returns
a dict of parameter transforms, which will be used if `make_safe = True`.

Example usage:

```
# Let's say we want a sample of size (batch_size, 10)
shapes = MultiVariateNormalDiag.param_shapes([batch_size, 10])

# shapes has a Tensor shape for mu and sigma
# shapes == {
#   "mu": tf.constant([batch_size, 10]),
#   "sigma": tf.constant([batch_size, 10]),
# }

# Here we parameterize mu and sigma with the output of a linear
# layer. Note that sigma is unconstrained.
params = {}
for name, shape in shapes.items():
  params[name] = linear(x, shape[1])

# Note that you can forward other kwargs to the `Distribution`, like
# `allow_nan_stats` or `name`.
mvn = MultiVariateNormalDiag.from_params(**params, allow_nan_stats=True)
```

Distribution parameters may have constraints (e.g. `sigma` must be positive
for a `Normal` distribution) and the `from_params` method will apply default
parameter transforms. If a user wants to use their own transform, they can
apply it externally and set `make_safe=False`.

##### Args:


*  <b>`make_safe`</b>: Whether the `params` should be constrained. If True,
    `from_params` will apply default parameter transforms. If False, no
    parameter transforms will be applied.
*  <b>`**kwargs`</b>: dict of parameters for the distribution.

##### Returns:

  A distribution parameterized by possibly transformed parameters in
  `kwargs`.

##### Raises:


*  <b>`TypeError`</b>: if `make_safe` is `True` but `_safe_transforms` is not
    implemented directly for `cls`.


- - -

#### `tf.contrib.distributions.MultivariateNormalFull.get_batch_shape()` {#MultivariateNormalFull.get_batch_shape}

Shape of a single sample from a single event index as a `TensorShape`.

Same meaning as `batch_shape`. May be only partially defined.

##### Returns:


*  <b>`batch_shape`</b>: `TensorShape`, possibly unknown.


- - -

#### `tf.contrib.distributions.MultivariateNormalFull.get_event_shape()` {#MultivariateNormalFull.get_event_shape}

Shape of a single sample from a single batch as a `TensorShape`.

Same meaning as `event_shape`. May be only partially defined.

##### Returns:


*  <b>`event_shape`</b>: `TensorShape`, possibly unknown.


- - -

#### `tf.contrib.distributions.MultivariateNormalFull.is_continuous` {#MultivariateNormalFull.is_continuous}




- - -

#### `tf.contrib.distributions.MultivariateNormalFull.is_reparameterized` {#MultivariateNormalFull.is_reparameterized}




- - -

#### `tf.contrib.distributions.MultivariateNormalFull.log_cdf(value, name='log_cdf')` {#MultivariateNormalFull.log_cdf}

Log cumulative distribution function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`logcdf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.MultivariateNormalFull.log_pdf(value, name='log_pdf')` {#MultivariateNormalFull.log_pdf}

Log probability density function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if not `is_continuous`.


- - -

#### `tf.contrib.distributions.MultivariateNormalFull.log_pmf(value, name='log_pmf')` {#MultivariateNormalFull.log_pmf}

Log probability mass function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_pmf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if `is_continuous`.


- - -

#### `tf.contrib.distributions.MultivariateNormalFull.log_prob(value, name='log_prob')` {#MultivariateNormalFull.log_prob}

Log probability density/mass function (depending on `is_continuous`).

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.MultivariateNormalFull.log_sigma_det(name='log_sigma_det')` {#MultivariateNormalFull.log_sigma_det}

Log of determinant of covariance matrix.


- - -

#### `tf.contrib.distributions.MultivariateNormalFull.mean(name='mean')` {#MultivariateNormalFull.mean}

Mean.


- - -

#### `tf.contrib.distributions.MultivariateNormalFull.mode(name='mode')` {#MultivariateNormalFull.mode}

Mode.


- - -

#### `tf.contrib.distributions.MultivariateNormalFull.mu` {#MultivariateNormalFull.mu}




- - -

#### `tf.contrib.distributions.MultivariateNormalFull.name` {#MultivariateNormalFull.name}

Name prepended to all ops created by this `Distribution`.


- - -

#### `tf.contrib.distributions.MultivariateNormalFull.param_shapes(cls, sample_shape, name='DistributionParamShapes')` {#MultivariateNormalFull.param_shapes}

Shapes of parameters given the desired shape of a call to `sample()`.

Subclasses should override static method `_param_shapes`.

##### Args:


*  <b>`sample_shape`</b>: `Tensor` or python list/tuple. Desired shape of a call to
    `sample()`.
*  <b>`name`</b>: name to prepend ops with.

##### Returns:

  `dict` of parameter name to `Tensor` shapes.


- - -

#### `tf.contrib.distributions.MultivariateNormalFull.param_static_shapes(cls, sample_shape)` {#MultivariateNormalFull.param_static_shapes}

param_shapes with static (i.e. TensorShape) shapes.

##### Args:


*  <b>`sample_shape`</b>: `TensorShape` or python list/tuple. Desired shape of a call
    to `sample()`.

##### Returns:

  `dict` of parameter name to `TensorShape`.

##### Raises:


*  <b>`ValueError`</b>: if `sample_shape` is a `TensorShape` and is not fully defined.


- - -

#### `tf.contrib.distributions.MultivariateNormalFull.parameters` {#MultivariateNormalFull.parameters}

Dictionary of parameters used by this `Distribution`.


- - -

#### `tf.contrib.distributions.MultivariateNormalFull.pdf(value, name='pdf')` {#MultivariateNormalFull.pdf}

Probability density function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if not `is_continuous`.


- - -

#### `tf.contrib.distributions.MultivariateNormalFull.pmf(value, name='pmf')` {#MultivariateNormalFull.pmf}

Probability mass function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`pmf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if `is_continuous`.


- - -

#### `tf.contrib.distributions.MultivariateNormalFull.prob(value, name='prob')` {#MultivariateNormalFull.prob}

Probability density/mass function (depending on `is_continuous`).

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.MultivariateNormalFull.sample(sample_shape=(), seed=None, name='sample')` {#MultivariateNormalFull.sample}

Generate samples of the specified shape.

Note that a call to `sample()` without arguments will generate a single
sample.

##### Args:


*  <b>`sample_shape`</b>: 0D or 1D `int32` `Tensor`. Shape of the generated samples.
*  <b>`seed`</b>: Python integer seed for RNG
*  <b>`name`</b>: name to give to the op.

##### Returns:


*  <b>`samples`</b>: a `Tensor` with prepended dimensions `sample_shape`.


- - -

#### `tf.contrib.distributions.MultivariateNormalFull.sample_n(n, seed=None, name='sample_n')` {#MultivariateNormalFull.sample_n}

Generate `n` samples.

##### Args:


*  <b>`n`</b>: `Scalar` `Tensor` of type `int32` or `int64`, the number of
    observations to sample.
*  <b>`seed`</b>: Python integer seed for RNG
*  <b>`name`</b>: name to give to the op.

##### Returns:


*  <b>`samples`</b>: a `Tensor` with a prepended dimension (n,).

##### Raises:


*  <b>`TypeError`</b>: if `n` is not an integer type.


- - -

#### `tf.contrib.distributions.MultivariateNormalFull.sigma` {#MultivariateNormalFull.sigma}

Dense (batch) covariance matrix, if available.


- - -

#### `tf.contrib.distributions.MultivariateNormalFull.sigma_det(name='sigma_det')` {#MultivariateNormalFull.sigma_det}

Determinant of covariance matrix.


- - -

#### `tf.contrib.distributions.MultivariateNormalFull.std(name='std')` {#MultivariateNormalFull.std}

Standard deviation.


- - -

#### `tf.contrib.distributions.MultivariateNormalFull.validate_args` {#MultivariateNormalFull.validate_args}

Python boolean indicated possibly expensive checks are enabled.


- - -

#### `tf.contrib.distributions.MultivariateNormalFull.variance(name='variance')` {#MultivariateNormalFull.variance}

Variance.



- - -

### `class tf.contrib.distributions.MultivariateNormalCholesky` {#MultivariateNormalCholesky}

The multivariate normal distribution on `R^k`.

This distribution is defined by a 1-D mean `mu` and a Cholesky factor `chol`.
Providing the Cholesky factor allows for `O(k^2)` pdf evaluation and sampling,
and requires `O(k^2)` storage.

#### Mathematical details

The Cholesky factor `chol` defines the covariance matrix: `C = chol chol^T`.

The PDF of this distribution is then:

```
f(x) = (2 pi)^(-k/2) |det(C)|^(-1/2) exp(-1/2 (x - mu)^T C^{-1} (x - mu))
```

#### Examples

A single multi-variate Gaussian distribution is defined by a vector of means
of length `k`, and a covariance matrix of shape `k x k`.

Extra leading dimensions, if provided, allow for batches.

```python
# Initialize a single 3-variate Gaussian with diagonal covariance.
# Note, this would be more efficient with MultivariateNormalDiag.
mu = [1, 2, 3.]
chol = [[1, 0, 0], [0, 3, 0], [0, 0, 2]]
dist = tf.contrib.distributions.MultivariateNormalCholesky(mu, chol)

# Evaluate this on an observation in R^3, returning a scalar.
dist.pdf([-1, 0, 1])

# Initialize a batch of two 3-variate Gaussians.
mu = [[1, 2, 3], [11, 22, 33]]
chol = ...  # shape 2 x 3 x 3, lower triangular, positive diagonal.
dist = tf.contrib.distributions.MultivariateNormalCholesky(mu, chol)

# Evaluate this on a two observations, each in R^3, returning a length two
# tensor.
x = [[-1, 0, 1], [-11, 0, 11]]  # Shape 2 x 3.
dist.pdf(x)
```

Trainable (batch) Choesky matrices can be created with
`tf.contrib.distributions.batch_matrix_diag_transform()`
- - -

#### `tf.contrib.distributions.MultivariateNormalCholesky.__init__(mu, chol, validate_args=True, allow_nan_stats=False, name='MultivariateNormalCholesky')` {#MultivariateNormalCholesky.__init__}

Multivariate Normal distributions on `R^k`.

User must provide means `mu` and `chol` which holds the (batch) Cholesky
factors, such that the covariance of each batch member is `chol chol^T`.

##### Args:


*  <b>`mu`</b>: `(N+1)-D` floating point tensor with shape `[N1,...,Nb, k]`,
    `b >= 0`.
*  <b>`chol`</b>: `(N+2)-D` `Tensor` with same `dtype` as `mu` and shape
    `[N1,...,Nb, k, k]`.  The upper triangular part is ignored (treated as
    though it is zero), and the diagonal must be positive.
*  <b>`validate_args`</b>: Whether to validate input with asserts.  If `validate_args`
    is `False`, and the inputs are invalid, correct behavior is not
    guaranteed.
*  <b>`allow_nan_stats`</b>: `Boolean`, default `False`.  If `False`, raise an
    exception if a statistic (e.g. mean/mode/etc...) is undefined for any
    batch member If `True`, batch members with valid parameters leading to
    undefined statistics will return NaN for this statistic.
*  <b>`name`</b>: The name to give Ops created by the initializer.

##### Raises:


*  <b>`TypeError`</b>: If `mu` and `chol` are different dtypes.


- - -

#### `tf.contrib.distributions.MultivariateNormalCholesky.allow_nan_stats` {#MultivariateNormalCholesky.allow_nan_stats}

Python boolean describing behavior when a stat is undefined.

Stats return +/- infinity when it makes sense.  E.g., the variance
of a Cauchy distribution is infinity.  However, sometimes the
statistic is undefined, e.g., if a distribution's pdf does not achieve a
maximum within the support of the distribution, the mode is undefined.
If the mean is undefined, then by definition the variance is undefined.
E.g. the mean for Student's T for df = 1 is undefined (no clear way to say
it is either + or - infinity), so the variance = E[(X - mean)^2] is also
undefined.

##### Returns:


*  <b>`allow_nan_stats`</b>: Python boolean.


- - -

#### `tf.contrib.distributions.MultivariateNormalCholesky.batch_shape(name='batch_shape')` {#MultivariateNormalCholesky.batch_shape}

Shape of a single sample from a single event index as a 1-D `Tensor`.

The product of the dimensions of the `batch_shape` is the number of
independent distributions of this kind the instance represents.

##### Args:


*  <b>`name`</b>: name to give to the op

##### Returns:


*  <b>`batch_shape`</b>: `Tensor`.


- - -

#### `tf.contrib.distributions.MultivariateNormalCholesky.cdf(value, name='cdf')` {#MultivariateNormalCholesky.cdf}

Cumulative distribution function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`cdf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.MultivariateNormalCholesky.dtype` {#MultivariateNormalCholesky.dtype}

The `DType` of `Tensor`s handled by this `Distribution`.


- - -

#### `tf.contrib.distributions.MultivariateNormalCholesky.entropy(name='entropy')` {#MultivariateNormalCholesky.entropy}

Shanon entropy in nats.


- - -

#### `tf.contrib.distributions.MultivariateNormalCholesky.event_shape(name='event_shape')` {#MultivariateNormalCholesky.event_shape}

Shape of a single sample from a single batch as a 1-D int32 `Tensor`.

##### Args:


*  <b>`name`</b>: name to give to the op

##### Returns:


*  <b>`event_shape`</b>: `Tensor`.


- - -

#### `tf.contrib.distributions.MultivariateNormalCholesky.from_params(cls, make_safe=True, **kwargs)` {#MultivariateNormalCholesky.from_params}

Given (unconstrained) parameters, return an instantiated distribution.

Subclasses should implement a static method `_safe_transforms` that returns
a dict of parameter transforms, which will be used if `make_safe = True`.

Example usage:

```
# Let's say we want a sample of size (batch_size, 10)
shapes = MultiVariateNormalDiag.param_shapes([batch_size, 10])

# shapes has a Tensor shape for mu and sigma
# shapes == {
#   "mu": tf.constant([batch_size, 10]),
#   "sigma": tf.constant([batch_size, 10]),
# }

# Here we parameterize mu and sigma with the output of a linear
# layer. Note that sigma is unconstrained.
params = {}
for name, shape in shapes.items():
  params[name] = linear(x, shape[1])

# Note that you can forward other kwargs to the `Distribution`, like
# `allow_nan_stats` or `name`.
mvn = MultiVariateNormalDiag.from_params(**params, allow_nan_stats=True)
```

Distribution parameters may have constraints (e.g. `sigma` must be positive
for a `Normal` distribution) and the `from_params` method will apply default
parameter transforms. If a user wants to use their own transform, they can
apply it externally and set `make_safe=False`.

##### Args:


*  <b>`make_safe`</b>: Whether the `params` should be constrained. If True,
    `from_params` will apply default parameter transforms. If False, no
    parameter transforms will be applied.
*  <b>`**kwargs`</b>: dict of parameters for the distribution.

##### Returns:

  A distribution parameterized by possibly transformed parameters in
  `kwargs`.

##### Raises:


*  <b>`TypeError`</b>: if `make_safe` is `True` but `_safe_transforms` is not
    implemented directly for `cls`.


- - -

#### `tf.contrib.distributions.MultivariateNormalCholesky.get_batch_shape()` {#MultivariateNormalCholesky.get_batch_shape}

Shape of a single sample from a single event index as a `TensorShape`.

Same meaning as `batch_shape`. May be only partially defined.

##### Returns:


*  <b>`batch_shape`</b>: `TensorShape`, possibly unknown.


- - -

#### `tf.contrib.distributions.MultivariateNormalCholesky.get_event_shape()` {#MultivariateNormalCholesky.get_event_shape}

Shape of a single sample from a single batch as a `TensorShape`.

Same meaning as `event_shape`. May be only partially defined.

##### Returns:


*  <b>`event_shape`</b>: `TensorShape`, possibly unknown.


- - -

#### `tf.contrib.distributions.MultivariateNormalCholesky.is_continuous` {#MultivariateNormalCholesky.is_continuous}




- - -

#### `tf.contrib.distributions.MultivariateNormalCholesky.is_reparameterized` {#MultivariateNormalCholesky.is_reparameterized}




- - -

#### `tf.contrib.distributions.MultivariateNormalCholesky.log_cdf(value, name='log_cdf')` {#MultivariateNormalCholesky.log_cdf}

Log cumulative distribution function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`logcdf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.MultivariateNormalCholesky.log_pdf(value, name='log_pdf')` {#MultivariateNormalCholesky.log_pdf}

Log probability density function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if not `is_continuous`.


- - -

#### `tf.contrib.distributions.MultivariateNormalCholesky.log_pmf(value, name='log_pmf')` {#MultivariateNormalCholesky.log_pmf}

Log probability mass function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_pmf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if `is_continuous`.


- - -

#### `tf.contrib.distributions.MultivariateNormalCholesky.log_prob(value, name='log_prob')` {#MultivariateNormalCholesky.log_prob}

Log probability density/mass function (depending on `is_continuous`).

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.MultivariateNormalCholesky.log_sigma_det(name='log_sigma_det')` {#MultivariateNormalCholesky.log_sigma_det}

Log of determinant of covariance matrix.


- - -

#### `tf.contrib.distributions.MultivariateNormalCholesky.mean(name='mean')` {#MultivariateNormalCholesky.mean}

Mean.


- - -

#### `tf.contrib.distributions.MultivariateNormalCholesky.mode(name='mode')` {#MultivariateNormalCholesky.mode}

Mode.


- - -

#### `tf.contrib.distributions.MultivariateNormalCholesky.mu` {#MultivariateNormalCholesky.mu}




- - -

#### `tf.contrib.distributions.MultivariateNormalCholesky.name` {#MultivariateNormalCholesky.name}

Name prepended to all ops created by this `Distribution`.


- - -

#### `tf.contrib.distributions.MultivariateNormalCholesky.param_shapes(cls, sample_shape, name='DistributionParamShapes')` {#MultivariateNormalCholesky.param_shapes}

Shapes of parameters given the desired shape of a call to `sample()`.

Subclasses should override static method `_param_shapes`.

##### Args:


*  <b>`sample_shape`</b>: `Tensor` or python list/tuple. Desired shape of a call to
    `sample()`.
*  <b>`name`</b>: name to prepend ops with.

##### Returns:

  `dict` of parameter name to `Tensor` shapes.


- - -

#### `tf.contrib.distributions.MultivariateNormalCholesky.param_static_shapes(cls, sample_shape)` {#MultivariateNormalCholesky.param_static_shapes}

param_shapes with static (i.e. TensorShape) shapes.

##### Args:


*  <b>`sample_shape`</b>: `TensorShape` or python list/tuple. Desired shape of a call
    to `sample()`.

##### Returns:

  `dict` of parameter name to `TensorShape`.

##### Raises:


*  <b>`ValueError`</b>: if `sample_shape` is a `TensorShape` and is not fully defined.


- - -

#### `tf.contrib.distributions.MultivariateNormalCholesky.parameters` {#MultivariateNormalCholesky.parameters}

Dictionary of parameters used by this `Distribution`.


- - -

#### `tf.contrib.distributions.MultivariateNormalCholesky.pdf(value, name='pdf')` {#MultivariateNormalCholesky.pdf}

Probability density function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if not `is_continuous`.


- - -

#### `tf.contrib.distributions.MultivariateNormalCholesky.pmf(value, name='pmf')` {#MultivariateNormalCholesky.pmf}

Probability mass function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`pmf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if `is_continuous`.


- - -

#### `tf.contrib.distributions.MultivariateNormalCholesky.prob(value, name='prob')` {#MultivariateNormalCholesky.prob}

Probability density/mass function (depending on `is_continuous`).

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.MultivariateNormalCholesky.sample(sample_shape=(), seed=None, name='sample')` {#MultivariateNormalCholesky.sample}

Generate samples of the specified shape.

Note that a call to `sample()` without arguments will generate a single
sample.

##### Args:


*  <b>`sample_shape`</b>: 0D or 1D `int32` `Tensor`. Shape of the generated samples.
*  <b>`seed`</b>: Python integer seed for RNG
*  <b>`name`</b>: name to give to the op.

##### Returns:


*  <b>`samples`</b>: a `Tensor` with prepended dimensions `sample_shape`.


- - -

#### `tf.contrib.distributions.MultivariateNormalCholesky.sample_n(n, seed=None, name='sample_n')` {#MultivariateNormalCholesky.sample_n}

Generate `n` samples.

##### Args:


*  <b>`n`</b>: `Scalar` `Tensor` of type `int32` or `int64`, the number of
    observations to sample.
*  <b>`seed`</b>: Python integer seed for RNG
*  <b>`name`</b>: name to give to the op.

##### Returns:


*  <b>`samples`</b>: a `Tensor` with a prepended dimension (n,).

##### Raises:


*  <b>`TypeError`</b>: if `n` is not an integer type.


- - -

#### `tf.contrib.distributions.MultivariateNormalCholesky.sigma` {#MultivariateNormalCholesky.sigma}

Dense (batch) covariance matrix, if available.


- - -

#### `tf.contrib.distributions.MultivariateNormalCholesky.sigma_det(name='sigma_det')` {#MultivariateNormalCholesky.sigma_det}

Determinant of covariance matrix.


- - -

#### `tf.contrib.distributions.MultivariateNormalCholesky.std(name='std')` {#MultivariateNormalCholesky.std}

Standard deviation.


- - -

#### `tf.contrib.distributions.MultivariateNormalCholesky.validate_args` {#MultivariateNormalCholesky.validate_args}

Python boolean indicated possibly expensive checks are enabled.


- - -

#### `tf.contrib.distributions.MultivariateNormalCholesky.variance(name='variance')` {#MultivariateNormalCholesky.variance}

Variance.



- - -

### `tf.contrib.distributions.batch_matrix_diag_transform(matrix, transform=None, name=None)` {#batch_matrix_diag_transform}

Transform diagonal of [batch-]matrix, leave rest of matrix unchanged.

Create a trainable covariance defined by a Cholesky factor:

```python
# Transform network layer into 2 x 2 array.
matrix_values = tf.contrib.layers.fully_connected(activations, 4)
matrix = tf.reshape(matrix_values, (batch_size, 2, 2))

# Make the diagonal positive.  If the upper triangle was zero, this would be a
# valid Cholesky factor.
chol = batch_matrix_diag_transform(matrix, transform=tf.nn.softplus)

# OperatorPDCholesky ignores the upper triangle.
operator = OperatorPDCholesky(chol)
```

Example of heteroskedastic 2-D linear regression.

```python
# Get a trainable Cholesky factor.
matrix_values = tf.contrib.layers.fully_connected(activations, 4)
matrix = tf.reshape(matrix_values, (batch_size, 2, 2))
chol = batch_matrix_diag_transform(matrix, transform=tf.nn.softplus)

# Get a trainable mean.
mu = tf.contrib.layers.fully_connected(activations, 2)

# This is a fully trainable multivariate normal!
dist = tf.contrib.distributions.MVNCholesky(mu, chol)

# Standard log loss.  Minimizing this will "train" mu and chol, and then dist
# will be a distribution predicting labels as multivariate Gaussians.
loss = -1 * tf.reduce_mean(dist.log_pdf(labels))
```

##### Args:


*  <b>`matrix`</b>: Rank `R` `Tensor`, `R >= 2`, where the last two dimensions are
    equal.
*  <b>`transform`</b>: Element-wise function mapping `Tensors` to `Tensors`.  To
    be applied to the diagonal of `matrix`.  If `None`, `matrix` is returned
    unchanged.  Defaults to `None`.
*  <b>`name`</b>: A name to give created ops.
    Defaults to "batch_matrix_diag_transform".

##### Returns:

  A `Tensor` with same shape and `dtype` as `matrix`.



#### Other multivariate distributions

- - -

### `class tf.contrib.distributions.Dirichlet` {#Dirichlet}

Dirichlet distribution.

This distribution is parameterized by a vector `alpha` of concentration
parameters for `k` classes.

#### Mathematical details

The Dirichlet is a distribution over the standard n-simplex, where the
standard n-simplex is defined by:
```{ (x_1, ..., x_n) in R^(n+1) | sum_j x_j = 1 and x_j >= 0 for all j }```.
The distribution has hyperparameters `alpha = (alpha_1,...,alpha_k)`,
and probability mass function (prob):

```prob(x) = 1 / Beta(alpha) * prod_j x_j^(alpha_j - 1)```

where `Beta(x) = prod_j Gamma(x_j) / Gamma(sum_j x_j)` is the multivariate
beta function.


This class provides methods to create indexed batches of Dirichlet
distributions.  If the provided `alpha` is rank 2 or higher, for
every fixed set of leading dimensions, the last dimension represents one
single Dirichlet distribution.  When calling distribution
functions (e.g. `dist.prob(x)`), `alpha` and `x` are broadcast to the
same shape (if possible).  In all cases, the last dimension of alpha/x
represents single Dirichlet distributions.

#### Examples

```python
alpha = [1, 2, 3]
dist = Dirichlet(alpha)
```

Creates a 3-class distribution, with the 3rd class is most likely to be drawn.
The distribution functions can be evaluated on x.

```python
# x same shape as alpha.
x = [.2, .3, .5]
dist.prob(x)  # Shape []

# alpha will be broadcast to [[1, 2, 3], [1, 2, 3]] to match x.
x = [[.1, .4, .5], [.2, .3, .5]]
dist.prob(x)  # Shape [2]

# alpha will be broadcast to shape [5, 7, 3] to match x.
x = [[...]]  # Shape [5, 7, 3]
dist.prob(x)  # Shape [5, 7]
```

Creates a 2-batch of 3-class distributions.

```python
alpha = [[1, 2, 3], [4, 5, 6]]  # Shape [2, 3]
dist = Dirichlet(alpha)

# x will be broadcast to [[2, 1, 0], [2, 1, 0]] to match alpha.
x = [.2, .3, .5]
dist.prob(x)  # Shape [2]
```
- - -

#### `tf.contrib.distributions.Dirichlet.__init__(alpha, validate_args=True, allow_nan_stats=False, name='Dirichlet')` {#Dirichlet.__init__}

Initialize a batch of Dirichlet distributions.

##### Args:


*  <b>`alpha`</b>: Positive floating point tensor with shape broadcastable to
    `[N1,..., Nm, k]` `m >= 0`.  Defines this as a batch of `N1 x ... x Nm`
     different `k` class Dirichlet distributions.
*  <b>`validate_args`</b>: Whether to assert valid values for parameters `alpha` and
    `x` in `prob` and `log_prob`.  If `False`, correct behavior is not
    guaranteed.
*  <b>`allow_nan_stats`</b>: Boolean, default `False`.  If `False`, raise an
    exception if a statistic (e.g. mean/mode/etc...) is undefined for any
    batch member.  If `True`, batch members with valid parameters leading to
    undefined statistics will return NaN for this statistic.
*  <b>`name`</b>: The name to prefix Ops created by this distribution class.


*  <b>`Examples`</b>: 

```python
# Define 1-batch of 2-class Dirichlet distributions,
# also known as a Beta distribution.
dist = Dirichlet([1.1, 2.0])

# Define a 2-batch of 3-class distributions.
dist = Dirichlet([[1.0, 2.0, 3.0], [4.0, 5.0, 6.0]])
```


- - -

#### `tf.contrib.distributions.Dirichlet.allow_nan_stats` {#Dirichlet.allow_nan_stats}

Python boolean describing behavior when a stat is undefined.

Stats return +/- infinity when it makes sense.  E.g., the variance
of a Cauchy distribution is infinity.  However, sometimes the
statistic is undefined, e.g., if a distribution's pdf does not achieve a
maximum within the support of the distribution, the mode is undefined.
If the mean is undefined, then by definition the variance is undefined.
E.g. the mean for Student's T for df = 1 is undefined (no clear way to say
it is either + or - infinity), so the variance = E[(X - mean)^2] is also
undefined.

##### Returns:


*  <b>`allow_nan_stats`</b>: Python boolean.


- - -

#### `tf.contrib.distributions.Dirichlet.alpha` {#Dirichlet.alpha}

Shape parameter.


- - -

#### `tf.contrib.distributions.Dirichlet.alpha_sum` {#Dirichlet.alpha_sum}

Sum of shape parameter.


- - -

#### `tf.contrib.distributions.Dirichlet.batch_shape(name='batch_shape')` {#Dirichlet.batch_shape}

Shape of a single sample from a single event index as a 1-D `Tensor`.

The product of the dimensions of the `batch_shape` is the number of
independent distributions of this kind the instance represents.

##### Args:


*  <b>`name`</b>: name to give to the op

##### Returns:


*  <b>`batch_shape`</b>: `Tensor`.


- - -

#### `tf.contrib.distributions.Dirichlet.cdf(value, name='cdf')` {#Dirichlet.cdf}

Cumulative distribution function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`cdf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Dirichlet.dtype` {#Dirichlet.dtype}

The `DType` of `Tensor`s handled by this `Distribution`.


- - -

#### `tf.contrib.distributions.Dirichlet.entropy(name='entropy')` {#Dirichlet.entropy}

Shanon entropy in nats.


- - -

#### `tf.contrib.distributions.Dirichlet.event_shape(name='event_shape')` {#Dirichlet.event_shape}

Shape of a single sample from a single batch as a 1-D int32 `Tensor`.

##### Args:


*  <b>`name`</b>: name to give to the op

##### Returns:


*  <b>`event_shape`</b>: `Tensor`.


- - -

#### `tf.contrib.distributions.Dirichlet.from_params(cls, make_safe=True, **kwargs)` {#Dirichlet.from_params}

Given (unconstrained) parameters, return an instantiated distribution.

Subclasses should implement a static method `_safe_transforms` that returns
a dict of parameter transforms, which will be used if `make_safe = True`.

Example usage:

```
# Let's say we want a sample of size (batch_size, 10)
shapes = MultiVariateNormalDiag.param_shapes([batch_size, 10])

# shapes has a Tensor shape for mu and sigma
# shapes == {
#   "mu": tf.constant([batch_size, 10]),
#   "sigma": tf.constant([batch_size, 10]),
# }

# Here we parameterize mu and sigma with the output of a linear
# layer. Note that sigma is unconstrained.
params = {}
for name, shape in shapes.items():
  params[name] = linear(x, shape[1])

# Note that you can forward other kwargs to the `Distribution`, like
# `allow_nan_stats` or `name`.
mvn = MultiVariateNormalDiag.from_params(**params, allow_nan_stats=True)
```

Distribution parameters may have constraints (e.g. `sigma` must be positive
for a `Normal` distribution) and the `from_params` method will apply default
parameter transforms. If a user wants to use their own transform, they can
apply it externally and set `make_safe=False`.

##### Args:


*  <b>`make_safe`</b>: Whether the `params` should be constrained. If True,
    `from_params` will apply default parameter transforms. If False, no
    parameter transforms will be applied.
*  <b>`**kwargs`</b>: dict of parameters for the distribution.

##### Returns:

  A distribution parameterized by possibly transformed parameters in
  `kwargs`.

##### Raises:


*  <b>`TypeError`</b>: if `make_safe` is `True` but `_safe_transforms` is not
    implemented directly for `cls`.


- - -

#### `tf.contrib.distributions.Dirichlet.get_batch_shape()` {#Dirichlet.get_batch_shape}

Shape of a single sample from a single event index as a `TensorShape`.

Same meaning as `batch_shape`. May be only partially defined.

##### Returns:


*  <b>`batch_shape`</b>: `TensorShape`, possibly unknown.


- - -

#### `tf.contrib.distributions.Dirichlet.get_event_shape()` {#Dirichlet.get_event_shape}

Shape of a single sample from a single batch as a `TensorShape`.

Same meaning as `event_shape`. May be only partially defined.

##### Returns:


*  <b>`event_shape`</b>: `TensorShape`, possibly unknown.


- - -

#### `tf.contrib.distributions.Dirichlet.is_continuous` {#Dirichlet.is_continuous}




- - -

#### `tf.contrib.distributions.Dirichlet.is_reparameterized` {#Dirichlet.is_reparameterized}




- - -

#### `tf.contrib.distributions.Dirichlet.log_cdf(value, name='log_cdf')` {#Dirichlet.log_cdf}

Log cumulative distribution function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`logcdf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Dirichlet.log_pdf(value, name='log_pdf')` {#Dirichlet.log_pdf}

Log probability density function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if not `is_continuous`.


- - -

#### `tf.contrib.distributions.Dirichlet.log_pmf(value, name='log_pmf')` {#Dirichlet.log_pmf}

Log probability mass function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_pmf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if `is_continuous`.


- - -

#### `tf.contrib.distributions.Dirichlet.log_prob(value, name='log_prob')` {#Dirichlet.log_prob}

Log probability density/mass function (depending on `is_continuous`).

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Dirichlet.mean(name='mean')` {#Dirichlet.mean}

Mean.


- - -

#### `tf.contrib.distributions.Dirichlet.mode(name='mode')` {#Dirichlet.mode}

Mode.


- - -

#### `tf.contrib.distributions.Dirichlet.name` {#Dirichlet.name}

Name prepended to all ops created by this `Distribution`.


- - -

#### `tf.contrib.distributions.Dirichlet.param_shapes(cls, sample_shape, name='DistributionParamShapes')` {#Dirichlet.param_shapes}

Shapes of parameters given the desired shape of a call to `sample()`.

Subclasses should override static method `_param_shapes`.

##### Args:


*  <b>`sample_shape`</b>: `Tensor` or python list/tuple. Desired shape of a call to
    `sample()`.
*  <b>`name`</b>: name to prepend ops with.

##### Returns:

  `dict` of parameter name to `Tensor` shapes.


- - -

#### `tf.contrib.distributions.Dirichlet.param_static_shapes(cls, sample_shape)` {#Dirichlet.param_static_shapes}

param_shapes with static (i.e. TensorShape) shapes.

##### Args:


*  <b>`sample_shape`</b>: `TensorShape` or python list/tuple. Desired shape of a call
    to `sample()`.

##### Returns:

  `dict` of parameter name to `TensorShape`.

##### Raises:


*  <b>`ValueError`</b>: if `sample_shape` is a `TensorShape` and is not fully defined.


- - -

#### `tf.contrib.distributions.Dirichlet.parameters` {#Dirichlet.parameters}

Dictionary of parameters used by this `Distribution`.


- - -

#### `tf.contrib.distributions.Dirichlet.pdf(value, name='pdf')` {#Dirichlet.pdf}

Probability density function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if not `is_continuous`.


- - -

#### `tf.contrib.distributions.Dirichlet.pmf(value, name='pmf')` {#Dirichlet.pmf}

Probability mass function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`pmf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if `is_continuous`.


- - -

#### `tf.contrib.distributions.Dirichlet.prob(value, name='prob')` {#Dirichlet.prob}

Probability density/mass function (depending on `is_continuous`).

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Dirichlet.sample(sample_shape=(), seed=None, name='sample')` {#Dirichlet.sample}

Generate samples of the specified shape.

Note that a call to `sample()` without arguments will generate a single
sample.

##### Args:


*  <b>`sample_shape`</b>: 0D or 1D `int32` `Tensor`. Shape of the generated samples.
*  <b>`seed`</b>: Python integer seed for RNG
*  <b>`name`</b>: name to give to the op.

##### Returns:


*  <b>`samples`</b>: a `Tensor` with prepended dimensions `sample_shape`.


- - -

#### `tf.contrib.distributions.Dirichlet.sample_n(n, seed=None, name='sample_n')` {#Dirichlet.sample_n}

Generate `n` samples.

##### Args:


*  <b>`n`</b>: `Scalar` `Tensor` of type `int32` or `int64`, the number of
    observations to sample.
*  <b>`seed`</b>: Python integer seed for RNG
*  <b>`name`</b>: name to give to the op.

##### Returns:


*  <b>`samples`</b>: a `Tensor` with a prepended dimension (n,).

##### Raises:


*  <b>`TypeError`</b>: if `n` is not an integer type.


- - -

#### `tf.contrib.distributions.Dirichlet.std(name='std')` {#Dirichlet.std}

Standard deviation.


- - -

#### `tf.contrib.distributions.Dirichlet.validate_args` {#Dirichlet.validate_args}

Python boolean indicated possibly expensive checks are enabled.


- - -

#### `tf.contrib.distributions.Dirichlet.variance(name='variance')` {#Dirichlet.variance}

Variance.



- - -

### `class tf.contrib.distributions.DirichletMultinomial` {#DirichletMultinomial}

DirichletMultinomial mixture distribution.

This distribution is parameterized by a vector `alpha` of concentration
parameters for `k` classes and `n`, the counts per each class..

#### Mathematical details

The Dirichlet Multinomial is a distribution over k-class count data, meaning
for each k-tuple of non-negative integer `counts = [c_1,...,c_k]`, we have a
probability of these draws being made from the distribution.  The distribution
has hyperparameters `alpha = (alpha_1,...,alpha_k)`, and probability mass
function (pmf):

```pmf(counts) = N! / (n_1!...n_k!) * Beta(alpha + c) / Beta(alpha)```

where above `N = sum_j n_j`, `N!` is `N` factorial, and
`Beta(x) = prod_j Gamma(x_j) / Gamma(sum_j x_j)` is the multivariate beta
function.

This is a mixture distribution in that `M` samples can be produced by:
  1. Choose class probabilities `p = (p_1,...,p_k) ~ Dir(alpha)`
  2. Draw integers `m = (n_1,...,n_k) ~ Multinomial(N, p)`

This class provides methods to create indexed batches of Dirichlet
Multinomial distributions.  If the provided `alpha` is rank 2 or higher, for
every fixed set of leading dimensions, the last dimension represents one
single Dirichlet Multinomial distribution.  When calling distribution
functions (e.g. `dist.pmf(counts)`), `alpha` and `counts` are broadcast to the
same shape (if possible).  In all cases, the last dimension of alpha/counts
represents single Dirichlet Multinomial distributions.

#### Examples

```python
alpha = [1, 2, 3]
n = 2
dist = DirichletMultinomial(n, alpha)
```

Creates a 3-class distribution, with the 3rd class is most likely to be drawn.
The distribution functions can be evaluated on counts.

```python
# counts same shape as alpha.
counts = [0, 0, 2]
dist.pmf(counts)  # Shape []

# alpha will be broadcast to [[1, 2, 3], [1, 2, 3]] to match counts.
counts = [[1, 1, 0], [1, 0, 1]]
dist.pmf(counts)  # Shape [2]

# alpha will be broadcast to shape [5, 7, 3] to match counts.
counts = [[...]]  # Shape [5, 7, 3]
dist.pmf(counts)  # Shape [5, 7]
```

Creates a 2-batch of 3-class distributions.

```python
alpha = [[1, 2, 3], [4, 5, 6]]  # Shape [2, 3]
n = [3, 3]
dist = DirichletMultinomial(n, alpha)

# counts will be broadcast to [[2, 1, 0], [2, 1, 0]] to match alpha.
counts = [2, 1, 0]
dist.pmf(counts)  # Shape [2]
```
- - -

#### `tf.contrib.distributions.DirichletMultinomial.__init__(n, alpha, validate_args=True, allow_nan_stats=False, name='DirichletMultinomial')` {#DirichletMultinomial.__init__}

Initialize a batch of DirichletMultinomial distributions.

##### Args:


*  <b>`n`</b>: Non-negative floating point tensor, whose dtype is the same as
    `alpha`. The shape is broadcastable to `[N1,..., Nm]` with `m >= 0`.
    Defines this as a batch of `N1 x ... x Nm` different Dirichlet
    multinomial distributions. Its components should be equal to integer
    values.
*  <b>`alpha`</b>: Positive floating point tensor, whose dtype is the same as
    `n` with shape broadcastable to `[N1,..., Nm, k]` `m >= 0`.  Defines
    this as a batch of `N1 x ... x Nm` different `k` class Dirichlet
    multinomial distributions.
*  <b>`validate_args`</b>: Whether to assert valid values for parameters `alpha` and
    `n`, and `x` in `prob` and `log_prob`.  If `False`, correct behavior is
    not guaranteed.
*  <b>`allow_nan_stats`</b>: Boolean, default `False`.  If `False`, raise an
    exception if a statistic (e.g. mean/mode/etc...) is undefined for any
    batch member.  If `True`, batch members with valid parameters leading to
    undefined statistics will return NaN for this statistic.
*  <b>`name`</b>: The name to prefix Ops created by this distribution class.


*  <b>`Examples`</b>: 

```python
# Define 1-batch of 2-class Dirichlet multinomial distribution,
# also known as a beta-binomial.
dist = DirichletMultinomial(2.0, [1.1, 2.0])

# Define a 2-batch of 3-class distributions.
dist = DirichletMultinomial([3., 4], [[1.0, 2.0, 3.0], [4.0, 5.0, 6.0]])
```


- - -

#### `tf.contrib.distributions.DirichletMultinomial.allow_nan_stats` {#DirichletMultinomial.allow_nan_stats}

Python boolean describing behavior when a stat is undefined.

Stats return +/- infinity when it makes sense.  E.g., the variance
of a Cauchy distribution is infinity.  However, sometimes the
statistic is undefined, e.g., if a distribution's pdf does not achieve a
maximum within the support of the distribution, the mode is undefined.
If the mean is undefined, then by definition the variance is undefined.
E.g. the mean for Student's T for df = 1 is undefined (no clear way to say
it is either + or - infinity), so the variance = E[(X - mean)^2] is also
undefined.

##### Returns:


*  <b>`allow_nan_stats`</b>: Python boolean.


- - -

#### `tf.contrib.distributions.DirichletMultinomial.alpha` {#DirichletMultinomial.alpha}

Parameter defining this distribution.


- - -

#### `tf.contrib.distributions.DirichletMultinomial.alpha_sum` {#DirichletMultinomial.alpha_sum}

Summation of alpha parameter.


- - -

#### `tf.contrib.distributions.DirichletMultinomial.batch_shape(name='batch_shape')` {#DirichletMultinomial.batch_shape}

Shape of a single sample from a single event index as a 1-D `Tensor`.

The product of the dimensions of the `batch_shape` is the number of
independent distributions of this kind the instance represents.

##### Args:


*  <b>`name`</b>: name to give to the op

##### Returns:


*  <b>`batch_shape`</b>: `Tensor`.


- - -

#### `tf.contrib.distributions.DirichletMultinomial.cdf(value, name='cdf')` {#DirichletMultinomial.cdf}

Cumulative distribution function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`cdf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.DirichletMultinomial.dtype` {#DirichletMultinomial.dtype}

The `DType` of `Tensor`s handled by this `Distribution`.


- - -

#### `tf.contrib.distributions.DirichletMultinomial.entropy(name='entropy')` {#DirichletMultinomial.entropy}

Shanon entropy in nats.


- - -

#### `tf.contrib.distributions.DirichletMultinomial.event_shape(name='event_shape')` {#DirichletMultinomial.event_shape}

Shape of a single sample from a single batch as a 1-D int32 `Tensor`.

##### Args:


*  <b>`name`</b>: name to give to the op

##### Returns:


*  <b>`event_shape`</b>: `Tensor`.


- - -

#### `tf.contrib.distributions.DirichletMultinomial.from_params(cls, make_safe=True, **kwargs)` {#DirichletMultinomial.from_params}

Given (unconstrained) parameters, return an instantiated distribution.

Subclasses should implement a static method `_safe_transforms` that returns
a dict of parameter transforms, which will be used if `make_safe = True`.

Example usage:

```
# Let's say we want a sample of size (batch_size, 10)
shapes = MultiVariateNormalDiag.param_shapes([batch_size, 10])

# shapes has a Tensor shape for mu and sigma
# shapes == {
#   "mu": tf.constant([batch_size, 10]),
#   "sigma": tf.constant([batch_size, 10]),
# }

# Here we parameterize mu and sigma with the output of a linear
# layer. Note that sigma is unconstrained.
params = {}
for name, shape in shapes.items():
  params[name] = linear(x, shape[1])

# Note that you can forward other kwargs to the `Distribution`, like
# `allow_nan_stats` or `name`.
mvn = MultiVariateNormalDiag.from_params(**params, allow_nan_stats=True)
```

Distribution parameters may have constraints (e.g. `sigma` must be positive
for a `Normal` distribution) and the `from_params` method will apply default
parameter transforms. If a user wants to use their own transform, they can
apply it externally and set `make_safe=False`.

##### Args:


*  <b>`make_safe`</b>: Whether the `params` should be constrained. If True,
    `from_params` will apply default parameter transforms. If False, no
    parameter transforms will be applied.
*  <b>`**kwargs`</b>: dict of parameters for the distribution.

##### Returns:

  A distribution parameterized by possibly transformed parameters in
  `kwargs`.

##### Raises:


*  <b>`TypeError`</b>: if `make_safe` is `True` but `_safe_transforms` is not
    implemented directly for `cls`.


- - -

#### `tf.contrib.distributions.DirichletMultinomial.get_batch_shape()` {#DirichletMultinomial.get_batch_shape}

Shape of a single sample from a single event index as a `TensorShape`.

Same meaning as `batch_shape`. May be only partially defined.

##### Returns:


*  <b>`batch_shape`</b>: `TensorShape`, possibly unknown.


- - -

#### `tf.contrib.distributions.DirichletMultinomial.get_event_shape()` {#DirichletMultinomial.get_event_shape}

Shape of a single sample from a single batch as a `TensorShape`.

Same meaning as `event_shape`. May be only partially defined.

##### Returns:


*  <b>`event_shape`</b>: `TensorShape`, possibly unknown.


- - -

#### `tf.contrib.distributions.DirichletMultinomial.is_continuous` {#DirichletMultinomial.is_continuous}




- - -

#### `tf.contrib.distributions.DirichletMultinomial.is_reparameterized` {#DirichletMultinomial.is_reparameterized}




- - -

#### `tf.contrib.distributions.DirichletMultinomial.log_cdf(value, name='log_cdf')` {#DirichletMultinomial.log_cdf}

Log cumulative distribution function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`logcdf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.DirichletMultinomial.log_pdf(value, name='log_pdf')` {#DirichletMultinomial.log_pdf}

Log probability density function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if not `is_continuous`.


- - -

#### `tf.contrib.distributions.DirichletMultinomial.log_pmf(value, name='log_pmf')` {#DirichletMultinomial.log_pmf}

Log probability mass function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_pmf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if `is_continuous`.


- - -

#### `tf.contrib.distributions.DirichletMultinomial.log_prob(value, name='log_prob')` {#DirichletMultinomial.log_prob}

Log probability density/mass function (depending on `is_continuous`).

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.DirichletMultinomial.mean(name='mean')` {#DirichletMultinomial.mean}

Mean.


- - -

#### `tf.contrib.distributions.DirichletMultinomial.mode(name='mode')` {#DirichletMultinomial.mode}

Mode.


- - -

#### `tf.contrib.distributions.DirichletMultinomial.n` {#DirichletMultinomial.n}

Parameter defining this distribution.


- - -

#### `tf.contrib.distributions.DirichletMultinomial.name` {#DirichletMultinomial.name}

Name prepended to all ops created by this `Distribution`.


- - -

#### `tf.contrib.distributions.DirichletMultinomial.param_shapes(cls, sample_shape, name='DistributionParamShapes')` {#DirichletMultinomial.param_shapes}

Shapes of parameters given the desired shape of a call to `sample()`.

Subclasses should override static method `_param_shapes`.

##### Args:


*  <b>`sample_shape`</b>: `Tensor` or python list/tuple. Desired shape of a call to
    `sample()`.
*  <b>`name`</b>: name to prepend ops with.

##### Returns:

  `dict` of parameter name to `Tensor` shapes.


- - -

#### `tf.contrib.distributions.DirichletMultinomial.param_static_shapes(cls, sample_shape)` {#DirichletMultinomial.param_static_shapes}

param_shapes with static (i.e. TensorShape) shapes.

##### Args:


*  <b>`sample_shape`</b>: `TensorShape` or python list/tuple. Desired shape of a call
    to `sample()`.

##### Returns:

  `dict` of parameter name to `TensorShape`.

##### Raises:


*  <b>`ValueError`</b>: if `sample_shape` is a `TensorShape` and is not fully defined.


- - -

#### `tf.contrib.distributions.DirichletMultinomial.parameters` {#DirichletMultinomial.parameters}

Dictionary of parameters used by this `Distribution`.


- - -

#### `tf.contrib.distributions.DirichletMultinomial.pdf(value, name='pdf')` {#DirichletMultinomial.pdf}

Probability density function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if not `is_continuous`.


- - -

#### `tf.contrib.distributions.DirichletMultinomial.pmf(value, name='pmf')` {#DirichletMultinomial.pmf}

Probability mass function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`pmf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if `is_continuous`.


- - -

#### `tf.contrib.distributions.DirichletMultinomial.prob(value, name='prob')` {#DirichletMultinomial.prob}

Probability density/mass function (depending on `is_continuous`).

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.DirichletMultinomial.sample(sample_shape=(), seed=None, name='sample')` {#DirichletMultinomial.sample}

Generate samples of the specified shape.

Note that a call to `sample()` without arguments will generate a single
sample.

##### Args:


*  <b>`sample_shape`</b>: 0D or 1D `int32` `Tensor`. Shape of the generated samples.
*  <b>`seed`</b>: Python integer seed for RNG
*  <b>`name`</b>: name to give to the op.

##### Returns:


*  <b>`samples`</b>: a `Tensor` with prepended dimensions `sample_shape`.


- - -

#### `tf.contrib.distributions.DirichletMultinomial.sample_n(n, seed=None, name='sample_n')` {#DirichletMultinomial.sample_n}

Generate `n` samples.

##### Args:


*  <b>`n`</b>: `Scalar` `Tensor` of type `int32` or `int64`, the number of
    observations to sample.
*  <b>`seed`</b>: Python integer seed for RNG
*  <b>`name`</b>: name to give to the op.

##### Returns:


*  <b>`samples`</b>: a `Tensor` with a prepended dimension (n,).

##### Raises:


*  <b>`TypeError`</b>: if `n` is not an integer type.


- - -

#### `tf.contrib.distributions.DirichletMultinomial.std(name='std')` {#DirichletMultinomial.std}

Standard deviation.


- - -

#### `tf.contrib.distributions.DirichletMultinomial.validate_args` {#DirichletMultinomial.validate_args}

Python boolean indicated possibly expensive checks are enabled.


- - -

#### `tf.contrib.distributions.DirichletMultinomial.variance(name='variance')` {#DirichletMultinomial.variance}

Variance.



- - -

### `class tf.contrib.distributions.Multinomial` {#Multinomial}

Multinomial distribution.

This distribution is parameterized by a vector `p` of probability
parameters for `k` classes and `n`, the counts per each class..

#### Mathematical details

The Multinomial is a distribution over k-class count data, meaning
for each k-tuple of non-negative integer `counts = [n_1,...,n_k]`, we have a
probability of these draws being made from the distribution.  The distribution
has hyperparameters `p = (p_1,...,p_k)`, and probability mass
function (pmf):

```pmf(counts) = n! / (n_1!...n_k!) * (p_1)^n_1*(p_2)^n_2*...(p_k)^n_k```

where above `n = sum_j n_j`, `n!` is `n` factorial.

#### Examples

Create a 3-class distribution, with the 3rd class is most likely to be drawn,
using logits..

```python
logits = [-50., -43, 0]
dist = Multinomial(n=4., logits=logits)
```

Create a 3-class distribution, with the 3rd class is most likely to be drawn.

```python
p = [.2, .3, .5]
dist = Multinomial(n=4., p=p)
```

The distribution functions can be evaluated on counts.

```python
# counts same shape as p.
counts = [1., 0, 3]
dist.prob(counts)  # Shape []

# p will be broadcast to [[.2, .3, .5], [.2, .3, .5]] to match counts.
counts = [[1., 2, 1], [2, 2, 0]]
dist.prob(counts)  # Shape [2]

# p will be broadcast to shape [5, 7, 3] to match counts.
counts = [[...]]  # Shape [5, 7, 3]
dist.prob(counts)  # Shape [5, 7]
```

Create a 2-batch of 3-class distributions.

```python
p = [[.1, .2, .7], [.3, .3, .4]]  # Shape [2, 3]
dist = Multinomial(n=[4., 5], p=p)

counts = [[2., 1, 1], [3, 1, 1]]
dist.prob(counts)  # Shape [2]
```
- - -

#### `tf.contrib.distributions.Multinomial.__init__(n, logits=None, p=None, validate_args=True, allow_nan_stats=False, name='Multinomial')` {#Multinomial.__init__}

Initialize a batch of Multinomial distributions.

##### Args:


*  <b>`n`</b>: Non-negative floating point tensor with shape broadcastable to
    `[N1,..., Nm]` with `m >= 0`. Defines this as a batch of
    `N1 x ... x Nm` different Multinomial distributions.  Its components
    should be equal to integer values.
*  <b>`logits`</b>: Floating point tensor representing the log-odds of a
    positive event with shape broadcastable to `[N1,..., Nm, k], m >= 0`,
    and the same dtype as `n`. Defines this as a batch of `N1 x ... x Nm`
    different `k` class Multinomial distributions.
*  <b>`p`</b>: Positive floating point tensor with shape broadcastable to
    `[N1,..., Nm, k]` `m >= 0` and same dtype as `n`.  Defines this as
    a batch of `N1 x ... x Nm` different `k` class Multinomial
    distributions. `p`'s components in the last portion of its shape should
    sum up to 1.
*  <b>`validate_args`</b>: Whether to assert valid values for parameters `n` and `p`,
    and `x` in `prob` and `log_prob`.  If `False`, correct behavior is not
    guaranteed.
*  <b>`allow_nan_stats`</b>: Boolean, default `False`.  If `False`, raise an
    exception if a statistic (e.g. mean/mode/etc...) is undefined for any
    batch member.  If `True`, batch members with valid parameters leading to
    undefined statistics will return NaN for this statistic.
*  <b>`name`</b>: The name to prefix Ops created by this distribution class.


*  <b>`Examples`</b>: 

```python
# Define 1-batch of 2-class multinomial distribution,
# also known as a Binomial distribution.
dist = Multinomial(n=2., p=[.1, .9])

# Define a 2-batch of 3-class distributions.
dist = Multinomial(n=[4., 5], p=[[.1, .3, .6], [.4, .05, .55]])
```


- - -

#### `tf.contrib.distributions.Multinomial.allow_nan_stats` {#Multinomial.allow_nan_stats}

Python boolean describing behavior when a stat is undefined.

Stats return +/- infinity when it makes sense.  E.g., the variance
of a Cauchy distribution is infinity.  However, sometimes the
statistic is undefined, e.g., if a distribution's pdf does not achieve a
maximum within the support of the distribution, the mode is undefined.
If the mean is undefined, then by definition the variance is undefined.
E.g. the mean for Student's T for df = 1 is undefined (no clear way to say
it is either + or - infinity), so the variance = E[(X - mean)^2] is also
undefined.

##### Returns:


*  <b>`allow_nan_stats`</b>: Python boolean.


- - -

#### `tf.contrib.distributions.Multinomial.batch_shape(name='batch_shape')` {#Multinomial.batch_shape}

Shape of a single sample from a single event index as a 1-D `Tensor`.

The product of the dimensions of the `batch_shape` is the number of
independent distributions of this kind the instance represents.

##### Args:


*  <b>`name`</b>: name to give to the op

##### Returns:


*  <b>`batch_shape`</b>: `Tensor`.


- - -

#### `tf.contrib.distributions.Multinomial.cdf(value, name='cdf')` {#Multinomial.cdf}

Cumulative distribution function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`cdf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Multinomial.dtype` {#Multinomial.dtype}

The `DType` of `Tensor`s handled by this `Distribution`.


- - -

#### `tf.contrib.distributions.Multinomial.entropy(name='entropy')` {#Multinomial.entropy}

Shanon entropy in nats.


- - -

#### `tf.contrib.distributions.Multinomial.event_shape(name='event_shape')` {#Multinomial.event_shape}

Shape of a single sample from a single batch as a 1-D int32 `Tensor`.

##### Args:


*  <b>`name`</b>: name to give to the op

##### Returns:


*  <b>`event_shape`</b>: `Tensor`.


- - -

#### `tf.contrib.distributions.Multinomial.from_params(cls, make_safe=True, **kwargs)` {#Multinomial.from_params}

Given (unconstrained) parameters, return an instantiated distribution.

Subclasses should implement a static method `_safe_transforms` that returns
a dict of parameter transforms, which will be used if `make_safe = True`.

Example usage:

```
# Let's say we want a sample of size (batch_size, 10)
shapes = MultiVariateNormalDiag.param_shapes([batch_size, 10])

# shapes has a Tensor shape for mu and sigma
# shapes == {
#   "mu": tf.constant([batch_size, 10]),
#   "sigma": tf.constant([batch_size, 10]),
# }

# Here we parameterize mu and sigma with the output of a linear
# layer. Note that sigma is unconstrained.
params = {}
for name, shape in shapes.items():
  params[name] = linear(x, shape[1])

# Note that you can forward other kwargs to the `Distribution`, like
# `allow_nan_stats` or `name`.
mvn = MultiVariateNormalDiag.from_params(**params, allow_nan_stats=True)
```

Distribution parameters may have constraints (e.g. `sigma` must be positive
for a `Normal` distribution) and the `from_params` method will apply default
parameter transforms. If a user wants to use their own transform, they can
apply it externally and set `make_safe=False`.

##### Args:


*  <b>`make_safe`</b>: Whether the `params` should be constrained. If True,
    `from_params` will apply default parameter transforms. If False, no
    parameter transforms will be applied.
*  <b>`**kwargs`</b>: dict of parameters for the distribution.

##### Returns:

  A distribution parameterized by possibly transformed parameters in
  `kwargs`.

##### Raises:


*  <b>`TypeError`</b>: if `make_safe` is `True` but `_safe_transforms` is not
    implemented directly for `cls`.


- - -

#### `tf.contrib.distributions.Multinomial.get_batch_shape()` {#Multinomial.get_batch_shape}

Shape of a single sample from a single event index as a `TensorShape`.

Same meaning as `batch_shape`. May be only partially defined.

##### Returns:


*  <b>`batch_shape`</b>: `TensorShape`, possibly unknown.


- - -

#### `tf.contrib.distributions.Multinomial.get_event_shape()` {#Multinomial.get_event_shape}

Shape of a single sample from a single batch as a `TensorShape`.

Same meaning as `event_shape`. May be only partially defined.

##### Returns:


*  <b>`event_shape`</b>: `TensorShape`, possibly unknown.


- - -

#### `tf.contrib.distributions.Multinomial.is_continuous` {#Multinomial.is_continuous}




- - -

#### `tf.contrib.distributions.Multinomial.is_reparameterized` {#Multinomial.is_reparameterized}




- - -

#### `tf.contrib.distributions.Multinomial.log_cdf(value, name='log_cdf')` {#Multinomial.log_cdf}

Log cumulative distribution function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`logcdf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Multinomial.log_pdf(value, name='log_pdf')` {#Multinomial.log_pdf}

Log probability density function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if not `is_continuous`.


- - -

#### `tf.contrib.distributions.Multinomial.log_pmf(value, name='log_pmf')` {#Multinomial.log_pmf}

Log probability mass function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_pmf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if `is_continuous`.


- - -

#### `tf.contrib.distributions.Multinomial.log_prob(value, name='log_prob')` {#Multinomial.log_prob}

Log probability density/mass function (depending on `is_continuous`).

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Multinomial.logits` {#Multinomial.logits}

Log-odds.


- - -

#### `tf.contrib.distributions.Multinomial.mean(name='mean')` {#Multinomial.mean}

Mean.


- - -

#### `tf.contrib.distributions.Multinomial.mode(name='mode')` {#Multinomial.mode}

Mode.


- - -

#### `tf.contrib.distributions.Multinomial.n` {#Multinomial.n}

Number of trials.


- - -

#### `tf.contrib.distributions.Multinomial.name` {#Multinomial.name}

Name prepended to all ops created by this `Distribution`.


- - -

#### `tf.contrib.distributions.Multinomial.p` {#Multinomial.p}

Event probabilities.


- - -

#### `tf.contrib.distributions.Multinomial.param_shapes(cls, sample_shape, name='DistributionParamShapes')` {#Multinomial.param_shapes}

Shapes of parameters given the desired shape of a call to `sample()`.

Subclasses should override static method `_param_shapes`.

##### Args:


*  <b>`sample_shape`</b>: `Tensor` or python list/tuple. Desired shape of a call to
    `sample()`.
*  <b>`name`</b>: name to prepend ops with.

##### Returns:

  `dict` of parameter name to `Tensor` shapes.


- - -

#### `tf.contrib.distributions.Multinomial.param_static_shapes(cls, sample_shape)` {#Multinomial.param_static_shapes}

param_shapes with static (i.e. TensorShape) shapes.

##### Args:


*  <b>`sample_shape`</b>: `TensorShape` or python list/tuple. Desired shape of a call
    to `sample()`.

##### Returns:

  `dict` of parameter name to `TensorShape`.

##### Raises:


*  <b>`ValueError`</b>: if `sample_shape` is a `TensorShape` and is not fully defined.


- - -

#### `tf.contrib.distributions.Multinomial.parameters` {#Multinomial.parameters}

Dictionary of parameters used by this `Distribution`.


- - -

#### `tf.contrib.distributions.Multinomial.pdf(value, name='pdf')` {#Multinomial.pdf}

Probability density function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if not `is_continuous`.


- - -

#### `tf.contrib.distributions.Multinomial.pmf(value, name='pmf')` {#Multinomial.pmf}

Probability mass function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`pmf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if `is_continuous`.


- - -

#### `tf.contrib.distributions.Multinomial.prob(value, name='prob')` {#Multinomial.prob}

Probability density/mass function (depending on `is_continuous`).

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.Multinomial.sample(sample_shape=(), seed=None, name='sample')` {#Multinomial.sample}

Generate samples of the specified shape.

Note that a call to `sample()` without arguments will generate a single
sample.

##### Args:


*  <b>`sample_shape`</b>: 0D or 1D `int32` `Tensor`. Shape of the generated samples.
*  <b>`seed`</b>: Python integer seed for RNG
*  <b>`name`</b>: name to give to the op.

##### Returns:


*  <b>`samples`</b>: a `Tensor` with prepended dimensions `sample_shape`.


- - -

#### `tf.contrib.distributions.Multinomial.sample_n(n, seed=None, name='sample_n')` {#Multinomial.sample_n}

Generate `n` samples.

##### Args:


*  <b>`n`</b>: `Scalar` `Tensor` of type `int32` or `int64`, the number of
    observations to sample.
*  <b>`seed`</b>: Python integer seed for RNG
*  <b>`name`</b>: name to give to the op.

##### Returns:


*  <b>`samples`</b>: a `Tensor` with a prepended dimension (n,).

##### Raises:


*  <b>`TypeError`</b>: if `n` is not an integer type.


- - -

#### `tf.contrib.distributions.Multinomial.std(name='std')` {#Multinomial.std}

Standard deviation.


- - -

#### `tf.contrib.distributions.Multinomial.validate_args` {#Multinomial.validate_args}

Python boolean indicated possibly expensive checks are enabled.


- - -

#### `tf.contrib.distributions.Multinomial.variance(name='variance')` {#Multinomial.variance}

Variance.



- - -

### `class tf.contrib.distributions.WishartCholesky` {#WishartCholesky}

The matrix Wishart distribution on positive definite matrices.

This distribution is defined by a scalar degrees of freedom `df` and a
lower, triangular Cholesky factor which characterizes the scale matrix.

Using WishartCholesky is a constant-time improvement over WishartFull. It
saves an O(nbk^3) operation, i.e., a matrix-product operation for sampling
and a Cholesky factorization in log_prob. For most use-cases it often saves
another O(nbk^3) operation since most uses of Wishart will also use the
Cholesky factorization.

#### Mathematical details.

The PDF of this distribution is,

```
f(X) = det(X)^(0.5 (df-k-1)) exp(-0.5 tr[inv(scale) X]) / B(scale, df)
```

where `df >= k` denotes the degrees of freedom, `scale` is a symmetric, pd,
`k x k` matrix, and the normalizing constant `B(scale, df)` is given by:

```
B(scale, df) = 2^(0.5 df k) |det(scale)|^(0.5 df) Gamma_k(0.5 df)
```

where `Gamma_k` is the multivariate Gamma function.


#### Examples

```python
# Initialize a single 3x3 Wishart with Cholesky factored scale matrix and 5
# degrees-of-freedom.(*)
df = 5
chol_scale = tf.cholesky(...)  # Shape is [3, 3].
dist = tf.contrib.distributions.WishartCholesky(df=df, scale=chol_scale)

# Evaluate this on an observation in R^3, returning a scalar.
x = ... # A 3x3 positive definite matrix.
dist.pdf(x)  # Shape is [], a scalar.

# Evaluate this on a two observations, each in R^{3x3}, returning a length two
# Tensor.
x = [x0, x1]  # Shape is [2, 3, 3].
dist.pdf(x)  # Shape is [2].

# Initialize two 3x3 Wisharts with Cholesky factored scale matrices.
df = [5, 4]
chol_scale = tf.batch_cholesky(...)  # Shape is [2, 3, 3].
dist = tf.contrib.distributions.WishartCholesky(df=df, scale=chol_scale)

# Evaluate this on four observations.
x = [[x0, x1], [x2, x3]]  # Shape is [2, 2, 3, 3].
dist.pdf(x)  # Shape is [2, 2].

# (*) - To efficiently create a trainable covariance matrix, see the example
#   in tf.contrib.distributions.batch_matrix_diag_transform.
```
- - -

#### `tf.contrib.distributions.WishartCholesky.__init__(df, scale, cholesky_input_output_matrices=False, validate_args=True, allow_nan_stats=False, name='WishartCholesky')` {#WishartCholesky.__init__}

Construct Wishart distributions.

##### Args:


*  <b>`df`</b>: `float` or `double` `Tensor`. Degrees of freedom, must be greater than
    or equal to dimension of the scale matrix.
*  <b>`scale`</b>: `float` or `double` `Tensor`. The Cholesky factorization of
    the symmetric positive definite scale matrix of the distribution.
*  <b>`cholesky_input_output_matrices`</b>: `Boolean`. Any function which whose input
    or output is a matrix assumes the input is Cholesky and returns a
    Cholesky factored matrix. Example`log_pdf` input takes a Cholesky and
    `sample_n` returns a Cholesky when
    `cholesky_input_output_matrices=True`.
*  <b>`validate_args`</b>: Whether to validate input with asserts. If `validate_args`
    is `False`, and the inputs are invalid, correct behavior is not
    guaranteed.
*  <b>`allow_nan_stats`</b>: `Boolean`, default `False`. If `False`, raise an
    exception if a statistic (e.g., mean, mode) is undefined for any batch
    member. If True, batch members with valid parameters leading to
    undefined statistics will return `NaN` for this statistic.
*  <b>`name`</b>: The name scope to give class member ops.


- - -

#### `tf.contrib.distributions.WishartCholesky.allow_nan_stats` {#WishartCholesky.allow_nan_stats}

Python boolean describing behavior when a stat is undefined.

Stats return +/- infinity when it makes sense.  E.g., the variance
of a Cauchy distribution is infinity.  However, sometimes the
statistic is undefined, e.g., if a distribution's pdf does not achieve a
maximum within the support of the distribution, the mode is undefined.
If the mean is undefined, then by definition the variance is undefined.
E.g. the mean for Student's T for df = 1 is undefined (no clear way to say
it is either + or - infinity), so the variance = E[(X - mean)^2] is also
undefined.

##### Returns:


*  <b>`allow_nan_stats`</b>: Python boolean.


- - -

#### `tf.contrib.distributions.WishartCholesky.batch_shape(name='batch_shape')` {#WishartCholesky.batch_shape}

Shape of a single sample from a single event index as a 1-D `Tensor`.

The product of the dimensions of the `batch_shape` is the number of
independent distributions of this kind the instance represents.

##### Args:


*  <b>`name`</b>: name to give to the op

##### Returns:


*  <b>`batch_shape`</b>: `Tensor`.


- - -

#### `tf.contrib.distributions.WishartCholesky.cdf(value, name='cdf')` {#WishartCholesky.cdf}

Cumulative distribution function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`cdf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.WishartCholesky.cholesky_input_output_matrices` {#WishartCholesky.cholesky_input_output_matrices}

Boolean indicating if `Tensor` input/outputs are Cholesky factorized.


- - -

#### `tf.contrib.distributions.WishartCholesky.df` {#WishartCholesky.df}

Wishart distribution degree(s) of freedom.


- - -

#### `tf.contrib.distributions.WishartCholesky.dimension` {#WishartCholesky.dimension}

Dimension of underlying vector space. The `p` in `R^(p*p)`.


- - -

#### `tf.contrib.distributions.WishartCholesky.dtype` {#WishartCholesky.dtype}

The `DType` of `Tensor`s handled by this `Distribution`.


- - -

#### `tf.contrib.distributions.WishartCholesky.entropy(name='entropy')` {#WishartCholesky.entropy}

Shanon entropy in nats.


- - -

#### `tf.contrib.distributions.WishartCholesky.event_shape(name='event_shape')` {#WishartCholesky.event_shape}

Shape of a single sample from a single batch as a 1-D int32 `Tensor`.

##### Args:


*  <b>`name`</b>: name to give to the op

##### Returns:


*  <b>`event_shape`</b>: `Tensor`.


- - -

#### `tf.contrib.distributions.WishartCholesky.from_params(cls, make_safe=True, **kwargs)` {#WishartCholesky.from_params}

Given (unconstrained) parameters, return an instantiated distribution.

Subclasses should implement a static method `_safe_transforms` that returns
a dict of parameter transforms, which will be used if `make_safe = True`.

Example usage:

```
# Let's say we want a sample of size (batch_size, 10)
shapes = MultiVariateNormalDiag.param_shapes([batch_size, 10])

# shapes has a Tensor shape for mu and sigma
# shapes == {
#   "mu": tf.constant([batch_size, 10]),
#   "sigma": tf.constant([batch_size, 10]),
# }

# Here we parameterize mu and sigma with the output of a linear
# layer. Note that sigma is unconstrained.
params = {}
for name, shape in shapes.items():
  params[name] = linear(x, shape[1])

# Note that you can forward other kwargs to the `Distribution`, like
# `allow_nan_stats` or `name`.
mvn = MultiVariateNormalDiag.from_params(**params, allow_nan_stats=True)
```

Distribution parameters may have constraints (e.g. `sigma` must be positive
for a `Normal` distribution) and the `from_params` method will apply default
parameter transforms. If a user wants to use their own transform, they can
apply it externally and set `make_safe=False`.

##### Args:


*  <b>`make_safe`</b>: Whether the `params` should be constrained. If True,
    `from_params` will apply default parameter transforms. If False, no
    parameter transforms will be applied.
*  <b>`**kwargs`</b>: dict of parameters for the distribution.

##### Returns:

  A distribution parameterized by possibly transformed parameters in
  `kwargs`.

##### Raises:


*  <b>`TypeError`</b>: if `make_safe` is `True` but `_safe_transforms` is not
    implemented directly for `cls`.


- - -

#### `tf.contrib.distributions.WishartCholesky.get_batch_shape()` {#WishartCholesky.get_batch_shape}

Shape of a single sample from a single event index as a `TensorShape`.

Same meaning as `batch_shape`. May be only partially defined.

##### Returns:


*  <b>`batch_shape`</b>: `TensorShape`, possibly unknown.


- - -

#### `tf.contrib.distributions.WishartCholesky.get_event_shape()` {#WishartCholesky.get_event_shape}

Shape of a single sample from a single batch as a `TensorShape`.

Same meaning as `event_shape`. May be only partially defined.

##### Returns:


*  <b>`event_shape`</b>: `TensorShape`, possibly unknown.


- - -

#### `tf.contrib.distributions.WishartCholesky.is_continuous` {#WishartCholesky.is_continuous}




- - -

#### `tf.contrib.distributions.WishartCholesky.is_reparameterized` {#WishartCholesky.is_reparameterized}




- - -

#### `tf.contrib.distributions.WishartCholesky.log_cdf(value, name='log_cdf')` {#WishartCholesky.log_cdf}

Log cumulative distribution function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`logcdf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.WishartCholesky.log_normalizing_constant(name='log_normalizing_constant')` {#WishartCholesky.log_normalizing_constant}

Computes the log normalizing constant, log(Z).


- - -

#### `tf.contrib.distributions.WishartCholesky.log_pdf(value, name='log_pdf')` {#WishartCholesky.log_pdf}

Log probability density function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if not `is_continuous`.


- - -

#### `tf.contrib.distributions.WishartCholesky.log_pmf(value, name='log_pmf')` {#WishartCholesky.log_pmf}

Log probability mass function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_pmf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if `is_continuous`.


- - -

#### `tf.contrib.distributions.WishartCholesky.log_prob(value, name='log_prob')` {#WishartCholesky.log_prob}

Log probability density/mass function (depending on `is_continuous`).

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.WishartCholesky.mean(name='mean')` {#WishartCholesky.mean}

Mean.


- - -

#### `tf.contrib.distributions.WishartCholesky.mean_log_det(name='mean_log_det')` {#WishartCholesky.mean_log_det}

Computes E[log(det(X))] under this Wishart distribution.


- - -

#### `tf.contrib.distributions.WishartCholesky.mode(name='mode')` {#WishartCholesky.mode}

Mode.


- - -

#### `tf.contrib.distributions.WishartCholesky.name` {#WishartCholesky.name}

Name prepended to all ops created by this `Distribution`.


- - -

#### `tf.contrib.distributions.WishartCholesky.param_shapes(cls, sample_shape, name='DistributionParamShapes')` {#WishartCholesky.param_shapes}

Shapes of parameters given the desired shape of a call to `sample()`.

Subclasses should override static method `_param_shapes`.

##### Args:


*  <b>`sample_shape`</b>: `Tensor` or python list/tuple. Desired shape of a call to
    `sample()`.
*  <b>`name`</b>: name to prepend ops with.

##### Returns:

  `dict` of parameter name to `Tensor` shapes.


- - -

#### `tf.contrib.distributions.WishartCholesky.param_static_shapes(cls, sample_shape)` {#WishartCholesky.param_static_shapes}

param_shapes with static (i.e. TensorShape) shapes.

##### Args:


*  <b>`sample_shape`</b>: `TensorShape` or python list/tuple. Desired shape of a call
    to `sample()`.

##### Returns:

  `dict` of parameter name to `TensorShape`.

##### Raises:


*  <b>`ValueError`</b>: if `sample_shape` is a `TensorShape` and is not fully defined.


- - -

#### `tf.contrib.distributions.WishartCholesky.parameters` {#WishartCholesky.parameters}

Dictionary of parameters used by this `Distribution`.


- - -

#### `tf.contrib.distributions.WishartCholesky.pdf(value, name='pdf')` {#WishartCholesky.pdf}

Probability density function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if not `is_continuous`.


- - -

#### `tf.contrib.distributions.WishartCholesky.pmf(value, name='pmf')` {#WishartCholesky.pmf}

Probability mass function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`pmf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if `is_continuous`.


- - -

#### `tf.contrib.distributions.WishartCholesky.prob(value, name='prob')` {#WishartCholesky.prob}

Probability density/mass function (depending on `is_continuous`).

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.WishartCholesky.sample(sample_shape=(), seed=None, name='sample')` {#WishartCholesky.sample}

Generate samples of the specified shape.

Note that a call to `sample()` without arguments will generate a single
sample.

##### Args:


*  <b>`sample_shape`</b>: 0D or 1D `int32` `Tensor`. Shape of the generated samples.
*  <b>`seed`</b>: Python integer seed for RNG
*  <b>`name`</b>: name to give to the op.

##### Returns:


*  <b>`samples`</b>: a `Tensor` with prepended dimensions `sample_shape`.


- - -

#### `tf.contrib.distributions.WishartCholesky.sample_n(n, seed=None, name='sample_n')` {#WishartCholesky.sample_n}

Generate `n` samples.

##### Args:


*  <b>`n`</b>: `Scalar` `Tensor` of type `int32` or `int64`, the number of
    observations to sample.
*  <b>`seed`</b>: Python integer seed for RNG
*  <b>`name`</b>: name to give to the op.

##### Returns:


*  <b>`samples`</b>: a `Tensor` with a prepended dimension (n,).

##### Raises:


*  <b>`TypeError`</b>: if `n` is not an integer type.


- - -

#### `tf.contrib.distributions.WishartCholesky.scale()` {#WishartCholesky.scale}

Wishart distribution scale matrix.


- - -

#### `tf.contrib.distributions.WishartCholesky.scale_operator_pd` {#WishartCholesky.scale_operator_pd}

Wishart distribution scale matrix as an OperatorPD.


- - -

#### `tf.contrib.distributions.WishartCholesky.std(name='std')` {#WishartCholesky.std}

Standard deviation.


- - -

#### `tf.contrib.distributions.WishartCholesky.validate_args` {#WishartCholesky.validate_args}

Python boolean indicated possibly expensive checks are enabled.


- - -

#### `tf.contrib.distributions.WishartCholesky.variance(name='variance')` {#WishartCholesky.variance}

Variance.



- - -

### `class tf.contrib.distributions.WishartFull` {#WishartFull}

The matrix Wishart distribution on positive definite matrices.

This distribution is defined by a scalar degrees of freedom `df` and a
symmetric, positive definite scale matrix.

Evaluation of the pdf, determinant, and sampling are all `O(k^3)` operations
where `(k, k)` is the event space shape.

#### Mathematical details.

The PDF of this distribution is,

```
f(X) = det(X)^(0.5 (df-k-1)) exp(-0.5 tr[inv(scale) X]) / B(scale, df)
```

where `df >= k` denotes the degrees of freedom, `scale` is a symmetric, pd,
`k x k` matrix, and the normalizing constant `B(scale, df)` is given by:

```
B(scale, df) = 2^(0.5 df k) |det(scale)|^(0.5 df) Gamma_k(0.5 df)
```

where `Gamma_k` is the multivariate Gamma function.

#### Examples

```python
# Initialize a single 3x3 Wishart with Full factored scale matrix and 5
# degrees-of-freedom.(*)
df = 5
scale = ...  # Shape is [3, 3]; positive definite.
dist = tf.contrib.distributions.WishartFull(df=df, scale=scale)

# Evaluate this on an observation in R^3, returning a scalar.
x = ... # A 3x3 positive definite matrix.
dist.pdf(x)  # Shape is [], a scalar.

# Evaluate this on a two observations, each in R^{3x3}, returning a length two
# Tensor.
x = [x0, x1]  # Shape is [2, 3, 3].
dist.pdf(x)  # Shape is [2].

# Initialize two 3x3 Wisharts with Full factored scale matrices.
df = [5, 4]
scale = ...  # Shape is [2, 3, 3].
dist = tf.contrib.distributions.WishartFull(df=df, scale=scale)

# Evaluate this on four observations.
x = [[x0, x1], [x2, x3]]  # Shape is [2, 2, 3, 3]; xi is positive definite.
dist.pdf(x)  # Shape is [2, 2].

# (*) - To efficiently create a trainable covariance matrix, see the example
#   in tf.contrib.distributions.batch_matrix_diag_transform.
```
- - -

#### `tf.contrib.distributions.WishartFull.__init__(df, scale, cholesky_input_output_matrices=False, validate_args=True, allow_nan_stats=False, name='WishartFull')` {#WishartFull.__init__}

Construct Wishart distributions.

##### Args:


*  <b>`df`</b>: `float` or `double` `Tensor`. Degrees of freedom, must be greater than
    or equal to dimension of the scale matrix.
*  <b>`scale`</b>: `float` or `double` `Tensor`. The symmetric positive definite
    scale matrix of the distribution.
*  <b>`cholesky_input_output_matrices`</b>: `Boolean`. Any function which whose input
    or output is a matrix assumes the input is Cholesky and returns a
    Cholesky factored matrix. Example`log_pdf` input takes a Cholesky and
    `sample_n` returns a Cholesky when
    `cholesky_input_output_matrices=True`.
*  <b>`validate_args`</b>: Whether to validate input with asserts. If `validate_args`
    is `False`, and the inputs are invalid, correct behavior is not
    guaranteed.
*  <b>`allow_nan_stats`</b>: `Boolean`, default `False`. If `False`, raise an
    exception if a statistic (e.g., mean, mode) is undefined for any batch
    member. If True, batch members with valid parameters leading to
    undefined statistics will return `NaN` for this statistic.
*  <b>`name`</b>: The name scope to give class member ops.


- - -

#### `tf.contrib.distributions.WishartFull.allow_nan_stats` {#WishartFull.allow_nan_stats}

Python boolean describing behavior when a stat is undefined.

Stats return +/- infinity when it makes sense.  E.g., the variance
of a Cauchy distribution is infinity.  However, sometimes the
statistic is undefined, e.g., if a distribution's pdf does not achieve a
maximum within the support of the distribution, the mode is undefined.
If the mean is undefined, then by definition the variance is undefined.
E.g. the mean for Student's T for df = 1 is undefined (no clear way to say
it is either + or - infinity), so the variance = E[(X - mean)^2] is also
undefined.

##### Returns:


*  <b>`allow_nan_stats`</b>: Python boolean.


- - -

#### `tf.contrib.distributions.WishartFull.batch_shape(name='batch_shape')` {#WishartFull.batch_shape}

Shape of a single sample from a single event index as a 1-D `Tensor`.

The product of the dimensions of the `batch_shape` is the number of
independent distributions of this kind the instance represents.

##### Args:


*  <b>`name`</b>: name to give to the op

##### Returns:


*  <b>`batch_shape`</b>: `Tensor`.


- - -

#### `tf.contrib.distributions.WishartFull.cdf(value, name='cdf')` {#WishartFull.cdf}

Cumulative distribution function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`cdf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.WishartFull.cholesky_input_output_matrices` {#WishartFull.cholesky_input_output_matrices}

Boolean indicating if `Tensor` input/outputs are Cholesky factorized.


- - -

#### `tf.contrib.distributions.WishartFull.df` {#WishartFull.df}

Wishart distribution degree(s) of freedom.


- - -

#### `tf.contrib.distributions.WishartFull.dimension` {#WishartFull.dimension}

Dimension of underlying vector space. The `p` in `R^(p*p)`.


- - -

#### `tf.contrib.distributions.WishartFull.dtype` {#WishartFull.dtype}

The `DType` of `Tensor`s handled by this `Distribution`.


- - -

#### `tf.contrib.distributions.WishartFull.entropy(name='entropy')` {#WishartFull.entropy}

Shanon entropy in nats.


- - -

#### `tf.contrib.distributions.WishartFull.event_shape(name='event_shape')` {#WishartFull.event_shape}

Shape of a single sample from a single batch as a 1-D int32 `Tensor`.

##### Args:


*  <b>`name`</b>: name to give to the op

##### Returns:


*  <b>`event_shape`</b>: `Tensor`.


- - -

#### `tf.contrib.distributions.WishartFull.from_params(cls, make_safe=True, **kwargs)` {#WishartFull.from_params}

Given (unconstrained) parameters, return an instantiated distribution.

Subclasses should implement a static method `_safe_transforms` that returns
a dict of parameter transforms, which will be used if `make_safe = True`.

Example usage:

```
# Let's say we want a sample of size (batch_size, 10)
shapes = MultiVariateNormalDiag.param_shapes([batch_size, 10])

# shapes has a Tensor shape for mu and sigma
# shapes == {
#   "mu": tf.constant([batch_size, 10]),
#   "sigma": tf.constant([batch_size, 10]),
# }

# Here we parameterize mu and sigma with the output of a linear
# layer. Note that sigma is unconstrained.
params = {}
for name, shape in shapes.items():
  params[name] = linear(x, shape[1])

# Note that you can forward other kwargs to the `Distribution`, like
# `allow_nan_stats` or `name`.
mvn = MultiVariateNormalDiag.from_params(**params, allow_nan_stats=True)
```

Distribution parameters may have constraints (e.g. `sigma` must be positive
for a `Normal` distribution) and the `from_params` method will apply default
parameter transforms. If a user wants to use their own transform, they can
apply it externally and set `make_safe=False`.

##### Args:


*  <b>`make_safe`</b>: Whether the `params` should be constrained. If True,
    `from_params` will apply default parameter transforms. If False, no
    parameter transforms will be applied.
*  <b>`**kwargs`</b>: dict of parameters for the distribution.

##### Returns:

  A distribution parameterized by possibly transformed parameters in
  `kwargs`.

##### Raises:


*  <b>`TypeError`</b>: if `make_safe` is `True` but `_safe_transforms` is not
    implemented directly for `cls`.


- - -

#### `tf.contrib.distributions.WishartFull.get_batch_shape()` {#WishartFull.get_batch_shape}

Shape of a single sample from a single event index as a `TensorShape`.

Same meaning as `batch_shape`. May be only partially defined.

##### Returns:


*  <b>`batch_shape`</b>: `TensorShape`, possibly unknown.


- - -

#### `tf.contrib.distributions.WishartFull.get_event_shape()` {#WishartFull.get_event_shape}

Shape of a single sample from a single batch as a `TensorShape`.

Same meaning as `event_shape`. May be only partially defined.

##### Returns:


*  <b>`event_shape`</b>: `TensorShape`, possibly unknown.


- - -

#### `tf.contrib.distributions.WishartFull.is_continuous` {#WishartFull.is_continuous}




- - -

#### `tf.contrib.distributions.WishartFull.is_reparameterized` {#WishartFull.is_reparameterized}




- - -

#### `tf.contrib.distributions.WishartFull.log_cdf(value, name='log_cdf')` {#WishartFull.log_cdf}

Log cumulative distribution function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`logcdf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.WishartFull.log_normalizing_constant(name='log_normalizing_constant')` {#WishartFull.log_normalizing_constant}

Computes the log normalizing constant, log(Z).


- - -

#### `tf.contrib.distributions.WishartFull.log_pdf(value, name='log_pdf')` {#WishartFull.log_pdf}

Log probability density function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if not `is_continuous`.


- - -

#### `tf.contrib.distributions.WishartFull.log_pmf(value, name='log_pmf')` {#WishartFull.log_pmf}

Log probability mass function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_pmf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if `is_continuous`.


- - -

#### `tf.contrib.distributions.WishartFull.log_prob(value, name='log_prob')` {#WishartFull.log_prob}

Log probability density/mass function (depending on `is_continuous`).

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.WishartFull.mean(name='mean')` {#WishartFull.mean}

Mean.


- - -

#### `tf.contrib.distributions.WishartFull.mean_log_det(name='mean_log_det')` {#WishartFull.mean_log_det}

Computes E[log(det(X))] under this Wishart distribution.


- - -

#### `tf.contrib.distributions.WishartFull.mode(name='mode')` {#WishartFull.mode}

Mode.


- - -

#### `tf.contrib.distributions.WishartFull.name` {#WishartFull.name}

Name prepended to all ops created by this `Distribution`.


- - -

#### `tf.contrib.distributions.WishartFull.param_shapes(cls, sample_shape, name='DistributionParamShapes')` {#WishartFull.param_shapes}

Shapes of parameters given the desired shape of a call to `sample()`.

Subclasses should override static method `_param_shapes`.

##### Args:


*  <b>`sample_shape`</b>: `Tensor` or python list/tuple. Desired shape of a call to
    `sample()`.
*  <b>`name`</b>: name to prepend ops with.

##### Returns:

  `dict` of parameter name to `Tensor` shapes.


- - -

#### `tf.contrib.distributions.WishartFull.param_static_shapes(cls, sample_shape)` {#WishartFull.param_static_shapes}

param_shapes with static (i.e. TensorShape) shapes.

##### Args:


*  <b>`sample_shape`</b>: `TensorShape` or python list/tuple. Desired shape of a call
    to `sample()`.

##### Returns:

  `dict` of parameter name to `TensorShape`.

##### Raises:


*  <b>`ValueError`</b>: if `sample_shape` is a `TensorShape` and is not fully defined.


- - -

#### `tf.contrib.distributions.WishartFull.parameters` {#WishartFull.parameters}

Dictionary of parameters used by this `Distribution`.


- - -

#### `tf.contrib.distributions.WishartFull.pdf(value, name='pdf')` {#WishartFull.pdf}

Probability density function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if not `is_continuous`.


- - -

#### `tf.contrib.distributions.WishartFull.pmf(value, name='pmf')` {#WishartFull.pmf}

Probability mass function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`pmf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if `is_continuous`.


- - -

#### `tf.contrib.distributions.WishartFull.prob(value, name='prob')` {#WishartFull.prob}

Probability density/mass function (depending on `is_continuous`).

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.WishartFull.sample(sample_shape=(), seed=None, name='sample')` {#WishartFull.sample}

Generate samples of the specified shape.

Note that a call to `sample()` without arguments will generate a single
sample.

##### Args:


*  <b>`sample_shape`</b>: 0D or 1D `int32` `Tensor`. Shape of the generated samples.
*  <b>`seed`</b>: Python integer seed for RNG
*  <b>`name`</b>: name to give to the op.

##### Returns:


*  <b>`samples`</b>: a `Tensor` with prepended dimensions `sample_shape`.


- - -

#### `tf.contrib.distributions.WishartFull.sample_n(n, seed=None, name='sample_n')` {#WishartFull.sample_n}

Generate `n` samples.

##### Args:


*  <b>`n`</b>: `Scalar` `Tensor` of type `int32` or `int64`, the number of
    observations to sample.
*  <b>`seed`</b>: Python integer seed for RNG
*  <b>`name`</b>: name to give to the op.

##### Returns:


*  <b>`samples`</b>: a `Tensor` with a prepended dimension (n,).

##### Raises:


*  <b>`TypeError`</b>: if `n` is not an integer type.


- - -

#### `tf.contrib.distributions.WishartFull.scale()` {#WishartFull.scale}

Wishart distribution scale matrix.


- - -

#### `tf.contrib.distributions.WishartFull.scale_operator_pd` {#WishartFull.scale_operator_pd}

Wishart distribution scale matrix as an OperatorPD.


- - -

#### `tf.contrib.distributions.WishartFull.std(name='std')` {#WishartFull.std}

Standard deviation.


- - -

#### `tf.contrib.distributions.WishartFull.validate_args` {#WishartFull.validate_args}

Python boolean indicated possibly expensive checks are enabled.


- - -

#### `tf.contrib.distributions.WishartFull.variance(name='variance')` {#WishartFull.variance}

Variance.




### Transformed distributions

- - -

### `class tf.contrib.distributions.TransformedDistribution` {#TransformedDistribution}

A Transformed Distribution.

A Transformed Distribution models `p(y)` given a base distribution `p(x)`,
an invertible transform, `y = f(x)`, and the determinant of the Jacobian of
`f(x)`.

Shapes, type, and reparameterization are taken from the base distribution.

#### Mathematical details

* `p(x)` - probability distribution for random variable X
* `p(y)` - probability distribution for random variable Y
* `f` - transform
* `g` - inverse transform, `g(f(x)) = x`
* `J(x)` - Jacobian of f(x)

A Transformed Distribution exposes `sample` and `pdf`:

  * `sample`: `y = f(x)`, after drawing a sample of X.
  * `pdf`: `p(y) = p(x) / det|J(x)| = p(g(y)) / det|J(g(y))|`

A simple example constructing a Log-Normal distribution from a Normal
distribution:

```
logit_normal = TransformedDistribution(
  base_dist=Normal(mu, sigma),
  transform=lambda x: tf.sigmoid(x),
  inverse=lambda y: tf.log(y) - tf.log(1. - y),
  log_det_jacobian=(lambda x:
      tf.reduce_sum(tf.log(tf.sigmoid(x)) + tf.log(1. - tf.sigmoid(x)),
                    reduction_indices=[-1])))
  name="LogitNormalTransformedDistribution"
)
```
- - -

#### `tf.contrib.distributions.TransformedDistribution.__init__(base_dist_cls, transform, inverse, log_det_jacobian, name='TransformedDistribution', **base_dist_args)` {#TransformedDistribution.__init__}

Construct a Transformed Distribution.

##### Args:


*  <b>`base_dist_cls`</b>: the base distribution class to transform. Must be a
      subclass of `Distribution`.
*  <b>`transform`</b>: a callable that takes a `Tensor` sample from `base_dist` and
      returns a `Tensor` of the same shape and type. `x => y`.
*  <b>`inverse`</b>: a callable that computes the inverse of transform. `y => x`. If
      None, users can only call `log_pdf` on values returned by `sample`.
*  <b>`log_det_jacobian`</b>: a callable that takes a `Tensor` sample from `base_dist`
      and returns the log of the determinant of the Jacobian of `transform`.
*  <b>`name`</b>: The name for the distribution.
*  <b>`**base_dist_args`</b>: kwargs to pass on to dist_cls on construction.

##### Raises:


*  <b>`TypeError`</b>: if `base_dist_cls` is not a subclass of
      `Distribution`.


- - -

#### `tf.contrib.distributions.TransformedDistribution.allow_nan_stats` {#TransformedDistribution.allow_nan_stats}

Python boolean describing behavior when a stat is undefined.

Stats return +/- infinity when it makes sense.  E.g., the variance
of a Cauchy distribution is infinity.  However, sometimes the
statistic is undefined, e.g., if a distribution's pdf does not achieve a
maximum within the support of the distribution, the mode is undefined.
If the mean is undefined, then by definition the variance is undefined.
E.g. the mean for Student's T for df = 1 is undefined (no clear way to say
it is either + or - infinity), so the variance = E[(X - mean)^2] is also
undefined.

##### Returns:


*  <b>`allow_nan_stats`</b>: Python boolean.


- - -

#### `tf.contrib.distributions.TransformedDistribution.base_distribution` {#TransformedDistribution.base_distribution}

Base distribution, p(x).


- - -

#### `tf.contrib.distributions.TransformedDistribution.batch_shape(name='batch_shape')` {#TransformedDistribution.batch_shape}

Shape of a single sample from a single event index as a 1-D `Tensor`.

The product of the dimensions of the `batch_shape` is the number of
independent distributions of this kind the instance represents.

##### Args:


*  <b>`name`</b>: name to give to the op

##### Returns:


*  <b>`batch_shape`</b>: `Tensor`.


- - -

#### `tf.contrib.distributions.TransformedDistribution.cdf(value, name='cdf')` {#TransformedDistribution.cdf}

Cumulative distribution function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`cdf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.TransformedDistribution.dtype` {#TransformedDistribution.dtype}

The `DType` of `Tensor`s handled by this `Distribution`.


- - -

#### `tf.contrib.distributions.TransformedDistribution.entropy(name='entropy')` {#TransformedDistribution.entropy}

Shanon entropy in nats.


- - -

#### `tf.contrib.distributions.TransformedDistribution.event_shape(name='event_shape')` {#TransformedDistribution.event_shape}

Shape of a single sample from a single batch as a 1-D int32 `Tensor`.

##### Args:


*  <b>`name`</b>: name to give to the op

##### Returns:


*  <b>`event_shape`</b>: `Tensor`.


- - -

#### `tf.contrib.distributions.TransformedDistribution.from_params(cls, make_safe=True, **kwargs)` {#TransformedDistribution.from_params}

Given (unconstrained) parameters, return an instantiated distribution.

Subclasses should implement a static method `_safe_transforms` that returns
a dict of parameter transforms, which will be used if `make_safe = True`.

Example usage:

```
# Let's say we want a sample of size (batch_size, 10)
shapes = MultiVariateNormalDiag.param_shapes([batch_size, 10])

# shapes has a Tensor shape for mu and sigma
# shapes == {
#   "mu": tf.constant([batch_size, 10]),
#   "sigma": tf.constant([batch_size, 10]),
# }

# Here we parameterize mu and sigma with the output of a linear
# layer. Note that sigma is unconstrained.
params = {}
for name, shape in shapes.items():
  params[name] = linear(x, shape[1])

# Note that you can forward other kwargs to the `Distribution`, like
# `allow_nan_stats` or `name`.
mvn = MultiVariateNormalDiag.from_params(**params, allow_nan_stats=True)
```

Distribution parameters may have constraints (e.g. `sigma` must be positive
for a `Normal` distribution) and the `from_params` method will apply default
parameter transforms. If a user wants to use their own transform, they can
apply it externally and set `make_safe=False`.

##### Args:


*  <b>`make_safe`</b>: Whether the `params` should be constrained. If True,
    `from_params` will apply default parameter transforms. If False, no
    parameter transforms will be applied.
*  <b>`**kwargs`</b>: dict of parameters for the distribution.

##### Returns:

  A distribution parameterized by possibly transformed parameters in
  `kwargs`.

##### Raises:


*  <b>`TypeError`</b>: if `make_safe` is `True` but `_safe_transforms` is not
    implemented directly for `cls`.


- - -

#### `tf.contrib.distributions.TransformedDistribution.get_batch_shape()` {#TransformedDistribution.get_batch_shape}

Shape of a single sample from a single event index as a `TensorShape`.

Same meaning as `batch_shape`. May be only partially defined.

##### Returns:


*  <b>`batch_shape`</b>: `TensorShape`, possibly unknown.


- - -

#### `tf.contrib.distributions.TransformedDistribution.get_event_shape()` {#TransformedDistribution.get_event_shape}

Shape of a single sample from a single batch as a `TensorShape`.

Same meaning as `event_shape`. May be only partially defined.

##### Returns:


*  <b>`event_shape`</b>: `TensorShape`, possibly unknown.


- - -

#### `tf.contrib.distributions.TransformedDistribution.inverse` {#TransformedDistribution.inverse}

Inverse function of transform, y => x.


- - -

#### `tf.contrib.distributions.TransformedDistribution.is_continuous` {#TransformedDistribution.is_continuous}




- - -

#### `tf.contrib.distributions.TransformedDistribution.is_reparameterized` {#TransformedDistribution.is_reparameterized}




- - -

#### `tf.contrib.distributions.TransformedDistribution.log_cdf(value, name='log_cdf')` {#TransformedDistribution.log_cdf}

Log cumulative distribution function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`logcdf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.TransformedDistribution.log_det_jacobian` {#TransformedDistribution.log_det_jacobian}

Function computing the log determinant of the Jacobian of transform.


- - -

#### `tf.contrib.distributions.TransformedDistribution.log_pdf(value, name='log_pdf')` {#TransformedDistribution.log_pdf}

Log probability density function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if not `is_continuous`.


- - -

#### `tf.contrib.distributions.TransformedDistribution.log_pmf(value, name='log_pmf')` {#TransformedDistribution.log_pmf}

Log probability mass function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_pmf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if `is_continuous`.


- - -

#### `tf.contrib.distributions.TransformedDistribution.log_prob(value, name='log_prob')` {#TransformedDistribution.log_prob}

Log probability density/mass function (depending on `is_continuous`).

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.TransformedDistribution.mean(name='mean')` {#TransformedDistribution.mean}

Mean.


- - -

#### `tf.contrib.distributions.TransformedDistribution.mode(name='mode')` {#TransformedDistribution.mode}

Mode.


- - -

#### `tf.contrib.distributions.TransformedDistribution.name` {#TransformedDistribution.name}

Name prepended to all ops created by this `Distribution`.


- - -

#### `tf.contrib.distributions.TransformedDistribution.param_shapes(cls, sample_shape, name='DistributionParamShapes')` {#TransformedDistribution.param_shapes}

Shapes of parameters given the desired shape of a call to `sample()`.

Subclasses should override static method `_param_shapes`.

##### Args:


*  <b>`sample_shape`</b>: `Tensor` or python list/tuple. Desired shape of a call to
    `sample()`.
*  <b>`name`</b>: name to prepend ops with.

##### Returns:

  `dict` of parameter name to `Tensor` shapes.


- - -

#### `tf.contrib.distributions.TransformedDistribution.param_static_shapes(cls, sample_shape)` {#TransformedDistribution.param_static_shapes}

param_shapes with static (i.e. TensorShape) shapes.

##### Args:


*  <b>`sample_shape`</b>: `TensorShape` or python list/tuple. Desired shape of a call
    to `sample()`.

##### Returns:

  `dict` of parameter name to `TensorShape`.

##### Raises:


*  <b>`ValueError`</b>: if `sample_shape` is a `TensorShape` and is not fully defined.


- - -

#### `tf.contrib.distributions.TransformedDistribution.parameters` {#TransformedDistribution.parameters}

Dictionary of parameters used by this `Distribution`.


- - -

#### `tf.contrib.distributions.TransformedDistribution.pdf(value, name='pdf')` {#TransformedDistribution.pdf}

Probability density function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if not `is_continuous`.


- - -

#### `tf.contrib.distributions.TransformedDistribution.pmf(value, name='pmf')` {#TransformedDistribution.pmf}

Probability mass function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`pmf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if `is_continuous`.


- - -

#### `tf.contrib.distributions.TransformedDistribution.prob(value, name='prob')` {#TransformedDistribution.prob}

Probability density/mass function (depending on `is_continuous`).

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.TransformedDistribution.sample(sample_shape=(), seed=None, name='sample')` {#TransformedDistribution.sample}

Generate samples of the specified shape.

Note that a call to `sample()` without arguments will generate a single
sample.

##### Args:


*  <b>`sample_shape`</b>: 0D or 1D `int32` `Tensor`. Shape of the generated samples.
*  <b>`seed`</b>: Python integer seed for RNG
*  <b>`name`</b>: name to give to the op.

##### Returns:


*  <b>`samples`</b>: a `Tensor` with prepended dimensions `sample_shape`.


- - -

#### `tf.contrib.distributions.TransformedDistribution.sample_n(n, seed=None, name='sample_n')` {#TransformedDistribution.sample_n}

Generate `n` samples.

##### Args:


*  <b>`n`</b>: `Scalar` `Tensor` of type `int32` or `int64`, the number of
    observations to sample.
*  <b>`seed`</b>: Python integer seed for RNG
*  <b>`name`</b>: name to give to the op.

##### Returns:


*  <b>`samples`</b>: a `Tensor` with a prepended dimension (n,).

##### Raises:


*  <b>`TypeError`</b>: if `n` is not an integer type.


- - -

#### `tf.contrib.distributions.TransformedDistribution.std(name='std')` {#TransformedDistribution.std}

Standard deviation.


- - -

#### `tf.contrib.distributions.TransformedDistribution.transform` {#TransformedDistribution.transform}

Function transforming x => y.


- - -

#### `tf.contrib.distributions.TransformedDistribution.validate_args` {#TransformedDistribution.validate_args}

Python boolean indicated possibly expensive checks are enabled.


- - -

#### `tf.contrib.distributions.TransformedDistribution.variance(name='variance')` {#TransformedDistribution.variance}

Variance.




## Posterior inference with conjugate priors.

Functions that transform conjugate prior/likelihood pairs to distributions
representing the posterior or posterior predictive.

### Normal likelihood with conjugate prior.

- - -

### `tf.contrib.distributions.normal_conjugates_known_sigma_posterior(prior, sigma, s, n)` {#normal_conjugates_known_sigma_posterior}

Posterior Normal distribution with conjugate prior on the mean.

This model assumes that `n` observations (with sum `s`) come from a
Normal with unknown mean `mu` (described by the Normal `prior`)
and known variance `sigma^2`.  The "known sigma posterior" is
the distribution of the unknown `mu`.

Accepts a prior Normal distribution object, having parameters
`mu0` and `sigma0`, as well as known `sigma` values of the predictive
distribution(s) (also assumed Normal),
and statistical estimates `s` (the sum(s) of the observations) and
`n` (the number(s) of observations).

Returns a posterior (also Normal) distribution object, with parameters
`(mu', sigma'^2)`, where:

```
mu ~ N(mu', sigma'^2)
sigma'^2 = 1/(1/sigma0^2 + n/sigma^2),
mu' = (mu0/sigma0^2 + s/sigma^2) * sigma'^2.
```

Distribution parameters from `prior`, as well as `sigma`, `s`, and `n`.
will broadcast in the case of multidimensional sets of parameters.

##### Args:


*  <b>`prior`</b>: `Normal` object of type `dtype`:
    the prior distribution having parameters `(mu0, sigma0)`.
*  <b>`sigma`</b>: tensor of type `dtype`, taking values `sigma > 0`.
    The known stddev parameter(s).
*  <b>`s`</b>: Tensor of type `dtype`.  The sum(s) of observations.
*  <b>`n`</b>: Tensor of type `int`.  The number(s) of observations.

##### Returns:

  A new Normal posterior distribution object for the unknown observation
  mean `mu`.

##### Raises:


*  <b>`TypeError`</b>: if dtype of `s` does not match `dtype`, or `prior` is not a
    Normal object.


- - -

### `tf.contrib.distributions.normal_congugates_known_sigma_predictive(prior, sigma, s, n)` {#normal_congugates_known_sigma_predictive}

Posterior predictive Normal distribution w. conjugate prior on the mean.

This model assumes that `n` observations (with sum `s`) come from a
Normal with unknown mean `mu` (described by the Normal `prior`)
and known variance `sigma^2`.  The "known sigma predictive"
is the distribution of new observations, conditioned on the existing
observations and our prior.

Accepts a prior Normal distribution object, having parameters
`mu0` and `sigma0`, as well as known `sigma` values of the predictive
distribution(s) (also assumed Normal),
and statistical estimates `s` (the sum(s) of the observations) and
`n` (the number(s) of observations).

Calculates the Normal distribution(s) `p(x | sigma^2)`:

```
  p(x | sigma^2) = int N(x | mu, sigma^2) N(mu | prior.mu, prior.sigma^2) dmu
                 = N(x | prior.mu, 1/(sigma^2 + prior.sigma^2))
```

Returns the predictive posterior distribution object, with parameters
`(mu', sigma'^2)`, where:

```
sigma_n^2 = 1/(1/sigma0^2 + n/sigma^2),
mu' = (mu0/sigma0^2 + s/sigma^2) * sigma_n^2.
sigma'^2 = sigma_n^2 + sigma^2,
```

Distribution parameters from `prior`, as well as `sigma`, `s`, and `n`.
will broadcast in the case of multidimensional sets of parameters.

##### Args:


*  <b>`prior`</b>: `Normal` object of type `dtype`:
    the prior distribution having parameters `(mu0, sigma0)`.
*  <b>`sigma`</b>: tensor of type `dtype`, taking values `sigma > 0`.
    The known stddev parameter(s).
*  <b>`s`</b>: Tensor of type `dtype`.  The sum(s) of observations.
*  <b>`n`</b>: Tensor of type `int`.  The number(s) of observations.

##### Returns:

  A new Normal predictive distribution object.

##### Raises:


*  <b>`TypeError`</b>: if dtype of `s` does not match `dtype`, or `prior` is not a
    Normal object.



## Kullback Leibler Divergence

- - -

### `tf.contrib.distributions.kl(dist_a, dist_b, allow_nan=False, name=None)` {#kl}

Get the KL-divergence KL(dist_a || dist_b).

##### Args:


*  <b>`dist_a`</b>: instance of distributions.Distribution.
*  <b>`dist_b`</b>: instance of distributions.Distribution.
*  <b>`allow_nan`</b>: If `False` (default), a runtime error is raised
    if the KL returns NaN values for any batch entry of the given
    distributions.  If `True`, the KL may return a NaN for the given entry.
*  <b>`name`</b>: (optional) Name scope to use for created operations.

##### Returns:

  A Tensor with the batchwise KL-divergence between dist_a and dist_b.

##### Raises:


*  <b>`TypeError`</b>: If dist_a or dist_b is not an instance of Distribution.
*  <b>`NotImplementedError`</b>: If no KL method is defined for distribution types
    of dist_a and dist_b.


- - -

### `class tf.contrib.distributions.RegisterKL` {#RegisterKL}

Decorator to register a KL divergence implementation function.

Usage:

@distributions.RegisterKL(distributions.Normal, distributions.Normal)
def _kl_normal_mvn(norm_a, norm_b):
  # Return KL(norm_a || norm_b)
- - -

#### `tf.contrib.distributions.RegisterKL.__init__(dist_cls_a, dist_cls_b)` {#RegisterKL.__init__}

Initialize the KL registrar.

##### Args:


*  <b>`dist_cls_a`</b>: the class of the first argument of the KL divergence.
*  <b>`dist_cls_b`</b>: the class of the second argument of the KL divergence.

##### Raises:


*  <b>`TypeError`</b>: if dist_cls_a or dist_cls_b are not subclasses of
    Distribution.




## Other Functions and Classes
- - -

### `class tf.contrib.distributions.BaseDistribution` {#BaseDistribution}

Simple abstract base class for probability distributions.

Implementations of core distributions to be included in the `distributions`
module should subclass `Distribution`. This base class may be useful to users
that want to fulfill a simpler distribution contract.
- - -

#### `tf.contrib.distributions.BaseDistribution.log_prob(value, name='log_prob')` {#BaseDistribution.log_prob}

Log probability density/mass function (depending on `is_continuous`).

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.BaseDistribution.sample_n(n, seed=None, name='sample')` {#BaseDistribution.sample_n}

Generate `n` samples.

##### Args:


*  <b>`n`</b>: `Scalar` `Tensor` of type `int32` or `int64`, the number of
    observations to sample.
*  <b>`seed`</b>: Python integer seed for RNG
*  <b>`name`</b>: name to give to the op.

##### Returns:


*  <b>`samples`</b>: a `Tensor` with a prepended dimension (n,).

##### Raises:


*  <b>`TypeError`</b>: if `n` is not an integer type.



- - -

### `class tf.contrib.distributions.MultivariateNormalDiagPlusVDVT` {#MultivariateNormalDiagPlusVDVT}

The multivariate normal distribution on `R^k`.

Every batch member of this distribution is defined by a mean and a lightweight
covariance matrix `C`.

#### Mathematical details

The PDF of this distribution in terms of the mean `mu` and covariance `C` is:

```
f(x) = (2 pi)^(-k/2) |det(C)|^(-1/2) exp(-1/2 (x - mu)^T C^{-1} (x - mu))
```

For every batch member, this distribution represents `k` random variables
`(X_1,...,X_k)`, with mean `E[X_i] = mu[i]`, and covariance matrix
`C_{ij} := E[(X_i - mu[i])(X_j - mu[j])]`

The user initializes this class by providing the mean `mu`, and a lightweight
definition of `C`:

```
C = SS^T = SS = (M + V D V^T) (M + V D V^T)
M is diagonal (k x k)
V = is shape (k x r), typically r << k
D = is diagonal (r x r), optional (defaults to identity).
```

This allows for `O(kr + r^3)` pdf evaluation and determinant, and `O(kr)`
sampling and storage (per batch member).

#### Examples

A single multi-variate Gaussian distribution is defined by a vector of means
of length `k`, and square root of the covariance `S = M + V D V^T`.  Extra
leading dimensions, if provided, allow for batches.

```python
# Initialize a single 3-variate Gaussian with covariance square root
# S = M + V D V^T, where V D V^T is a matrix-rank 2 update.
mu = [1, 2, 3.]
diag_large = [1.1, 2.2, 3.3]
v = ... # shape 3 x 2
diag_small = [4., 5.]
dist = tf.contrib.distributions.MultivariateNormalDiagPlusVDVT(
    mu, diag_large, v, diag_small=diag_small)

# Evaluate this on an observation in R^3, returning a scalar.
dist.pdf([-1, 0, 1])

# Initialize a batch of two 3-variate Gaussians.  This time, don't provide
# diag_small.  This means S = M + V V^T.
mu = [[1, 2, 3], [11, 22, 33]]  # shape 2 x 3
diag_large = ... # shape 2 x 3
v = ... # shape 2 x 3 x 1, a matrix-rank 1 update.
dist = tf.contrib.distributions.MultivariateNormalDiagPlusVDVT(
    mu, diag_large, v)

# Evaluate this on a two observations, each in R^3, returning a length two
# tensor.
x = [[-1, 0, 1], [-11, 0, 11]]  # Shape 2 x 3.
dist.pdf(x)
```
- - -

#### `tf.contrib.distributions.MultivariateNormalDiagPlusVDVT.__init__(mu, diag_large, v, diag_small=None, validate_args=True, allow_nan_stats=False, name='MultivariateNormalDiagPlusVDVT')` {#MultivariateNormalDiagPlusVDVT.__init__}

Multivariate Normal distributions on `R^k`.

For every batch member, this distribution represents `k` random variables
`(X_1,...,X_k)`, with mean `E[X_i] = mu[i]`, and covariance matrix
`C_{ij} := E[(X_i - mu[i])(X_j - mu[j])]`

The user initializes this class by providing the mean `mu`, and a
lightweight definition of `C`:

```
C = SS^T = SS = (M + V D V^T) (M + V D V^T)
M is diagonal (k x k)
V = is shape (k x r), typically r << k
D = is diagonal (r x r), optional (defaults to identity).
```

##### Args:


*  <b>`mu`</b>: Rank `n + 1` floating point tensor with shape `[N1,...,Nn, k]`,
    `n >= 0`.  The means.
*  <b>`diag_large`</b>: Optional rank `n + 1` floating point tensor, shape
    `[N1,...,Nn, k]` `n >= 0`.  Defines the diagonal matrix `M`.
*  <b>`v`</b>: Rank `n + 1` floating point tensor, shape `[N1,...,Nn, k, r]`
    `n >= 0`.  Defines the matrix `V`.
*  <b>`diag_small`</b>: Rank `n + 1` floating point tensor, shape
    `[N1,...,Nn, k]` `n >= 0`.  Defines the diagonal matrix `D`.  Default
    is `None`, which means `D` will be the identity matrix.
*  <b>`validate_args`</b>: Whether to validate input with asserts.  If `validate_args`
    is `False`,
    and the inputs are invalid, correct behavior is not guaranteed.
*  <b>`allow_nan_stats`</b>: `Boolean`, default `False`.  If `False`, raise an
    exception if a statistic (e.g. mean/mode/etc...) is undefined for any
    batch member If `True`, batch members with valid parameters leading to
    undefined statistics will return NaN for this statistic.
*  <b>`name`</b>: The name to give Ops created by the initializer.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiagPlusVDVT.allow_nan_stats` {#MultivariateNormalDiagPlusVDVT.allow_nan_stats}

Python boolean describing behavior when a stat is undefined.

Stats return +/- infinity when it makes sense.  E.g., the variance
of a Cauchy distribution is infinity.  However, sometimes the
statistic is undefined, e.g., if a distribution's pdf does not achieve a
maximum within the support of the distribution, the mode is undefined.
If the mean is undefined, then by definition the variance is undefined.
E.g. the mean for Student's T for df = 1 is undefined (no clear way to say
it is either + or - infinity), so the variance = E[(X - mean)^2] is also
undefined.

##### Returns:


*  <b>`allow_nan_stats`</b>: Python boolean.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiagPlusVDVT.batch_shape(name='batch_shape')` {#MultivariateNormalDiagPlusVDVT.batch_shape}

Shape of a single sample from a single event index as a 1-D `Tensor`.

The product of the dimensions of the `batch_shape` is the number of
independent distributions of this kind the instance represents.

##### Args:


*  <b>`name`</b>: name to give to the op

##### Returns:


*  <b>`batch_shape`</b>: `Tensor`.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiagPlusVDVT.cdf(value, name='cdf')` {#MultivariateNormalDiagPlusVDVT.cdf}

Cumulative distribution function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`cdf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiagPlusVDVT.dtype` {#MultivariateNormalDiagPlusVDVT.dtype}

The `DType` of `Tensor`s handled by this `Distribution`.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiagPlusVDVT.entropy(name='entropy')` {#MultivariateNormalDiagPlusVDVT.entropy}

Shanon entropy in nats.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiagPlusVDVT.event_shape(name='event_shape')` {#MultivariateNormalDiagPlusVDVT.event_shape}

Shape of a single sample from a single batch as a 1-D int32 `Tensor`.

##### Args:


*  <b>`name`</b>: name to give to the op

##### Returns:


*  <b>`event_shape`</b>: `Tensor`.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiagPlusVDVT.from_params(cls, make_safe=True, **kwargs)` {#MultivariateNormalDiagPlusVDVT.from_params}

Given (unconstrained) parameters, return an instantiated distribution.

Subclasses should implement a static method `_safe_transforms` that returns
a dict of parameter transforms, which will be used if `make_safe = True`.

Example usage:

```
# Let's say we want a sample of size (batch_size, 10)
shapes = MultiVariateNormalDiag.param_shapes([batch_size, 10])

# shapes has a Tensor shape for mu and sigma
# shapes == {
#   "mu": tf.constant([batch_size, 10]),
#   "sigma": tf.constant([batch_size, 10]),
# }

# Here we parameterize mu and sigma with the output of a linear
# layer. Note that sigma is unconstrained.
params = {}
for name, shape in shapes.items():
  params[name] = linear(x, shape[1])

# Note that you can forward other kwargs to the `Distribution`, like
# `allow_nan_stats` or `name`.
mvn = MultiVariateNormalDiag.from_params(**params, allow_nan_stats=True)
```

Distribution parameters may have constraints (e.g. `sigma` must be positive
for a `Normal` distribution) and the `from_params` method will apply default
parameter transforms. If a user wants to use their own transform, they can
apply it externally and set `make_safe=False`.

##### Args:


*  <b>`make_safe`</b>: Whether the `params` should be constrained. If True,
    `from_params` will apply default parameter transforms. If False, no
    parameter transforms will be applied.
*  <b>`**kwargs`</b>: dict of parameters for the distribution.

##### Returns:

  A distribution parameterized by possibly transformed parameters in
  `kwargs`.

##### Raises:


*  <b>`TypeError`</b>: if `make_safe` is `True` but `_safe_transforms` is not
    implemented directly for `cls`.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiagPlusVDVT.get_batch_shape()` {#MultivariateNormalDiagPlusVDVT.get_batch_shape}

Shape of a single sample from a single event index as a `TensorShape`.

Same meaning as `batch_shape`. May be only partially defined.

##### Returns:


*  <b>`batch_shape`</b>: `TensorShape`, possibly unknown.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiagPlusVDVT.get_event_shape()` {#MultivariateNormalDiagPlusVDVT.get_event_shape}

Shape of a single sample from a single batch as a `TensorShape`.

Same meaning as `event_shape`. May be only partially defined.

##### Returns:


*  <b>`event_shape`</b>: `TensorShape`, possibly unknown.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiagPlusVDVT.is_continuous` {#MultivariateNormalDiagPlusVDVT.is_continuous}




- - -

#### `tf.contrib.distributions.MultivariateNormalDiagPlusVDVT.is_reparameterized` {#MultivariateNormalDiagPlusVDVT.is_reparameterized}




- - -

#### `tf.contrib.distributions.MultivariateNormalDiagPlusVDVT.log_cdf(value, name='log_cdf')` {#MultivariateNormalDiagPlusVDVT.log_cdf}

Log cumulative distribution function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`logcdf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiagPlusVDVT.log_pdf(value, name='log_pdf')` {#MultivariateNormalDiagPlusVDVT.log_pdf}

Log probability density function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if not `is_continuous`.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiagPlusVDVT.log_pmf(value, name='log_pmf')` {#MultivariateNormalDiagPlusVDVT.log_pmf}

Log probability mass function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_pmf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if `is_continuous`.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiagPlusVDVT.log_prob(value, name='log_prob')` {#MultivariateNormalDiagPlusVDVT.log_prob}

Log probability density/mass function (depending on `is_continuous`).

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`log_prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiagPlusVDVT.log_sigma_det(name='log_sigma_det')` {#MultivariateNormalDiagPlusVDVT.log_sigma_det}

Log of determinant of covariance matrix.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiagPlusVDVT.mean(name='mean')` {#MultivariateNormalDiagPlusVDVT.mean}

Mean.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiagPlusVDVT.mode(name='mode')` {#MultivariateNormalDiagPlusVDVT.mode}

Mode.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiagPlusVDVT.mu` {#MultivariateNormalDiagPlusVDVT.mu}




- - -

#### `tf.contrib.distributions.MultivariateNormalDiagPlusVDVT.name` {#MultivariateNormalDiagPlusVDVT.name}

Name prepended to all ops created by this `Distribution`.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiagPlusVDVT.param_shapes(cls, sample_shape, name='DistributionParamShapes')` {#MultivariateNormalDiagPlusVDVT.param_shapes}

Shapes of parameters given the desired shape of a call to `sample()`.

Subclasses should override static method `_param_shapes`.

##### Args:


*  <b>`sample_shape`</b>: `Tensor` or python list/tuple. Desired shape of a call to
    `sample()`.
*  <b>`name`</b>: name to prepend ops with.

##### Returns:

  `dict` of parameter name to `Tensor` shapes.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiagPlusVDVT.param_static_shapes(cls, sample_shape)` {#MultivariateNormalDiagPlusVDVT.param_static_shapes}

param_shapes with static (i.e. TensorShape) shapes.

##### Args:


*  <b>`sample_shape`</b>: `TensorShape` or python list/tuple. Desired shape of a call
    to `sample()`.

##### Returns:

  `dict` of parameter name to `TensorShape`.

##### Raises:


*  <b>`ValueError`</b>: if `sample_shape` is a `TensorShape` and is not fully defined.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiagPlusVDVT.parameters` {#MultivariateNormalDiagPlusVDVT.parameters}

Dictionary of parameters used by this `Distribution`.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiagPlusVDVT.pdf(value, name='pdf')` {#MultivariateNormalDiagPlusVDVT.pdf}

Probability density function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if not `is_continuous`.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiagPlusVDVT.pmf(value, name='pmf')` {#MultivariateNormalDiagPlusVDVT.pmf}

Probability mass function.

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`pmf`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.

##### Raises:


*  <b>`AttributeError`</b>: if `is_continuous`.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiagPlusVDVT.prob(value, name='prob')` {#MultivariateNormalDiagPlusVDVT.prob}

Probability density/mass function (depending on `is_continuous`).

##### Args:


*  <b>`value`</b>: `float` or `double` `Tensor`.
*  <b>`name`</b>: The name to give this op.

##### Returns:


*  <b>`prob`</b>: a `Tensor` of shape `sample_shape(x) + self.batch_shape` with
    values of type `self.dtype`.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiagPlusVDVT.sample(sample_shape=(), seed=None, name='sample')` {#MultivariateNormalDiagPlusVDVT.sample}

Generate samples of the specified shape.

Note that a call to `sample()` without arguments will generate a single
sample.

##### Args:


*  <b>`sample_shape`</b>: 0D or 1D `int32` `Tensor`. Shape of the generated samples.
*  <b>`seed`</b>: Python integer seed for RNG
*  <b>`name`</b>: name to give to the op.

##### Returns:


*  <b>`samples`</b>: a `Tensor` with prepended dimensions `sample_shape`.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiagPlusVDVT.sample_n(n, seed=None, name='sample_n')` {#MultivariateNormalDiagPlusVDVT.sample_n}

Generate `n` samples.

##### Args:


*  <b>`n`</b>: `Scalar` `Tensor` of type `int32` or `int64`, the number of
    observations to sample.
*  <b>`seed`</b>: Python integer seed for RNG
*  <b>`name`</b>: name to give to the op.

##### Returns:


*  <b>`samples`</b>: a `Tensor` with a prepended dimension (n,).

##### Raises:


*  <b>`TypeError`</b>: if `n` is not an integer type.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiagPlusVDVT.sigma` {#MultivariateNormalDiagPlusVDVT.sigma}

Dense (batch) covariance matrix, if available.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiagPlusVDVT.sigma_det(name='sigma_det')` {#MultivariateNormalDiagPlusVDVT.sigma_det}

Determinant of covariance matrix.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiagPlusVDVT.std(name='std')` {#MultivariateNormalDiagPlusVDVT.std}

Standard deviation.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiagPlusVDVT.validate_args` {#MultivariateNormalDiagPlusVDVT.validate_args}

Python boolean indicated possibly expensive checks are enabled.


- - -

#### `tf.contrib.distributions.MultivariateNormalDiagPlusVDVT.variance(name='variance')` {#MultivariateNormalDiagPlusVDVT.variance}

Variance.



