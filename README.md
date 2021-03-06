# LombScargle

[![Build Status](https://travis-ci.org/giordano/LombScargle.jl.svg?branch=master)](https://travis-ci.org/giordano/LombScargle.jl) [![Build status](https://ci.appveyor.com/api/projects/status/vv6mho713fuse6qy/branch/master?svg=true)](https://ci.appveyor.com/project/giordano/lombscargle-jl/branch/master) [![Coverage Status](https://coveralls.io/repos/github/giordano/LombScargle.jl/badge.svg?branch=master)](https://coveralls.io/github/giordano/LombScargle.jl?branch=master) [![codecov](https://codecov.io/gh/giordano/LombScargle.jl/branch/master/graph/badge.svg)](https://codecov.io/gh/giordano/LombScargle.jl) [![LombScargle](http://pkg.julialang.org/badges/LombScargle_0.4.svg)](http://pkg.julialang.org/?pkg=LombScargle) [![LombScargle](http://pkg.julialang.org/badges/LombScargle_0.5.svg)](http://pkg.julialang.org/?pkg=LombScargle)

Introduction
------------

`LombScargle.jl` is a [Julia](http://julialang.org/) package to estimate the
[frequency spectrum](https://en.wikipedia.org/wiki/Frequency_spectrum) of a
periodic signal with
[the Lomb–Scargle periodogram](https://en.wikipedia.org/wiki/The_Lomb–Scargle_periodogram).

Another Julia package that provides tools to perform spectral analysis of
signals is [`DSP.jl`](https://github.com/JuliaDSP/DSP.jl), but its methods
require that the signal has been sampled at equally spaced times.  Instead, the
Lomb–Scargle periodogram enables you to analyze unevenly sampled data as well,
which is a fairly common case in astronomy.

The algorithm used in this package are reported in the following papers:

* Townsend, R. H. D. 2010, ApJS, 191, 247 (URL:
  http://dx.doi.org/10.1088/0067-0049/191/2/247, Bibcode:
  http://adsabs.harvard.edu/abs/2010ApJS..191..247T)
* Zechmeister, M., Kürster, M. 2009, A&A, 496, 577 (URL:
  http://dx.doi.org/10.1051/0004-6361:200811296, Bibcode:
  http://adsabs.harvard.edu/abs/2009A%26A...496..577Z)

### Documentation ###

The complete manual of `LombScargle.jl` is available at
http://lombscarglejl.readthedocs.io.  There, people interested in the details
of the package, in order integrate the package in their workflow, can can find a
technical appendix explaining how the package internally works.  You can also
download the PDF version of the manual from
https://media.readthedocs.org/pdf/lombscarglejl/latest/lombscarglejl.pdf.

Installation
------------

`LombScargle.jl` is available for Julia 0.4 and later versions, and can be
installed with
[Julia built-in package manager](http://docs.julialang.org/en/stable/manual/packages/).
In a Julia session run the command

```julia
julia> Pkg.add("LombScargle")
```

You may need to update your package list with `Pkg.update()` in order to get the
latest version of `LombScargle.jl`.

Usage
-----

After installing the package, you can start using it with

```julia
using LombScargle
```

The module defines a new `LombScargle.Periodogram` data type, which, however, is
not exported because you will most probably not need to directly manipulate
`LombScargle.Periodogram` objects.  This data type holds both the frequency and
the power vectors of the periodogram.

The main function provided by the package is `lombscargle`:

```julia
lombscargle(times::AbstractVector{Real}, signal::AbstractVector{Real},
            errors::AbstractVector{Real}=ones(signal);
            normalization::AbstractString="standard",
            noise_level::Real=1.0,
            center_data::Bool=true, fit_mean::Bool=true,
            samples_per_peak::Integer=5,
            nyquist_factor::Integer=5,
            minimum_frequency::Real=NaN,
            maximum_frequency::Real=NaN,
            frequencies::AbstractVector{Real}=
            autofrequency(times,
                          samples_per_peak=samples_per_peak,
                          nyquist_factor=nyquist_factor,
                          minimum_frequency=minimum_frequency,
                          maximum_frequency=maximum_frequency))
```

which returns a `LombScargle.Periodogram`.  The mandatory arguments are:

* `times`: the vector of observation times
* `signal`: the vector of observations associated with `times`

Optional argument is:

* `errors`: the uncertainties associated to each `signal` point

All the vectors above must have the same length.

Optional keyword arguments are:

* `normalization`: how to normalize the periodogram.  Valid choices are:
  `"standard"`, `"model"`, `"log"`, `"psd"`, `"Scargle"`, `"HorneBaliunas"`,
  `"Cumming"`
* `noise_level`: the noise level used to normalize the periodogram when
  `normalization` is set to `"Scargle"`
* `fit_mean`: if `true`, fit for the mean of the signal using the Generalised
  Lomb–Scargle algorithm (see Zechmeister & Kürster paper).  If this is `false`,
  the original algorithm by Lomb and Scargle will be employed (see Townsend
  paper), which does not take into account a non-null mean and uncertainties for
  observations
* `center_data`: if `true`, subtract the mean of `signal` from `signal` itself
  before performing the periodogram.  This is especially important if `fit_mean`
  is `false`
* `frequencies`: the frequecy grid (not angular frequencies) at which the
  periodogram will be computed, as a vector.  If not provided, it is
  automatically determined with `LombScargle.autofrequency` function, which see.
  See below for other available keywords that can be used to adjust the
  frequency grid without directly setting `frequencies`

In addition, you can use all optional keyword arguments of
`LombScargle.autofrequency` function in order to tune the `frequencies` vector
without calling the function:

* `samples_per_peak`: the approximate number of desired samples across the
  typical peak
* `nyquist_factor`: the multiple of the average Nyquist frequency used to choose
  the maximum frequency if `maximum_frequency` is not provided
* `minimum_frequency`: if specified, then use this minimum frequency rather than
  one chosen based on the size of the baseline
* `maximum_frequency`: if specified, then use this maximum frequency rather than
  one chosen based on the average Nyquist frequency

The frequency grid is determined by following prescriptions given at
https://jakevdp.github.io/blog/2015/06/13/lomb-scargle-in-python/ and uses the
same keywords names adopted in Astropy.

If the signal has uncertainties, the `signal` vector can also be a vector of
`Measurement` objects (from
[`Measurements.jl`](https://github.com/giordano/Measurements.jl) package), in
which case you don’t need to pass a separate `errors` vector for the
uncertainties of the signal.  You can create arrays of `Measurement` objects
with the `measurement` function, see `Measurements.jl` manual at
http://measurementsjl.readthedocs.io/ for more details.

### Access Frequency Grid and Power Spectrum of the Periodogram ###

```julia
power(p::Periodogram)
freq(p::Periodogram)
freqpower(p::Periodogram)
```

`lombscargle` function return a `LombScargle.Periodogram` object, but you most
probably want to use the frequency grid and the power spectrum.  You can access
these vectors with `freq` and `power` functions, just like in `DSP.jl` package.
If you want to get the 2-tuple `(freq(p), power(p))` use the `freqpower`
function.

### Find Highest Power and Associated Frequencies ###

```julia
findmaxpower(p::Periodogram)
findmaxfreq(p::Periodogram, threshold::Real=findmaxpower(p))
```

Once you compute the periodogram, you usually want to know which are the
frequencies with highest power.  To do this, you can use the `findmaxfreq`.  It
returns the vector of frequencies with the highest power in the periodogram `p`.
If a second argument `threshold` is provided, return the frequencies with power
larger than or equal to `threshold`.  The value of the highest power of a
periodogram can be calculated with the `findmaxpower` function.

Examples
--------

Here is an example of a noisy periodic signal (`sin(π*t) + 1.5*cos(2π*t)`)
sampled at unevenly spaced times.

```julia
using LombScargle
ntimes = 1001
# Observation times
t = linspace(0.01, 10pi, ntimes)
# Randomize times
t += step(t)*rand(ntimes)
# The signal
s = sinpi(t) + 1.5cospi(2t) + rand(ntimes)
pgram = lombscargle(t, s)
```

You can plot the result, for example with
[`PyPlot`](https://github.com/stevengj/PyPlot.jl) package.  Use `freqpower`
function to get the frequency grid and the power of the periodogram as a
2-tuple.

```julia
using PyPlot
plot(freqpower(pgram)...)
```

Beware that if you use original Lomb–Scargle algorithm (`fit_mean=false` keyword
to `lombscargle` function) without centering the data (`center_data=false`) you
can get inaccurate results.  For example, spurious peaks at low frequencies can
appear.

```julia
plot(freqpower(lombscargle(t, s, fit_mean=false, center_data=false))...)
```

You can tune the frequency grid with appropriate keywords to `lombscargle`
function.  For example, in order to increase the sampling increase
`samples_per_peak`, and set `maximum_frequency` to lower values in order to
narrow the frequency range:

```julia
plot(freqpower(lombscargle(t, s, samples_per_peak=20, maximum_frequency=1.5))...)
```

If you simply want to use your own frequency grid, directly set the
`frequencies` keyword:

```julia
plot(freqpower(lombscargle(t, s, frequencies=0.001:1e-3:1.5))...)
```

### Signal with Uncertainties ###

The generalised Lomb–Scargle periodogram (used when the `fit_mean` optional
keyword is `true`) is able to handle a signal with uncertainties, and they will
be used as weights in the algorithm.  The uncertainties can be passed either as
the third optional argument `errors` to `lombscargle` or by providing this
function with a `signal` vector of type `Measurement` (from
[`Measurements.jl`](https://github.com/giordano/Measurements.jl) package).

```julia
using Measurements, PyPlot
ntimes = 1001
t = linspace(0.01, 10pi, ntimes)
s = sinpi(2t)
errors = rand(0.1:1e-3:4.0, ntimes)
plot(freqpower(lombscargle(t, s, errors, maximum_frequency=1.5))...)
plot(freqpower(lombscargle(t, measurement(s, errors), maximum_frequency=1.5))...)
```

### `findmaxfreq` Function ###

`findmaxfreq` function tells you the frequencies with the highest power in the
periodogram (and you can get the period by taking its inverse):

```julia
t = linspace(0, 10, 1001)
s = sinpi(2t)
p = lombscargle(t, s)
1.0./findmaxfreq(p) # Period with highest power
# => 1-element Array{Float64,1}:
#     0.00502487
```

This peak is at high frequency, very far from the expected value of the period
of 1.  In order to find the real peak, you can either narrow the frequency range
in order to exclude higher armonics, or pass the `threshold` argument to
`findmaxfreq`.  You can use `findmaxpower` to discover the highest power in the
periodogram:

```julia
findmaxpower(p)
# => 0.9712085205753647
1.0./findmaxfreq(p, 0.97)
# => 5-element Array{Float64,1}:
#     1.0101
#     0.0101
#     0.00990197
#     0.00502487
#     0.00497537
```

The first peak is the real one, the other double peaks appear at higher
armonics.  Usually plotting the periodogram can give you a clue of what’s going
on.

Development
-----------

The package is developed at https://github.com/giordano/LombScargle.jl.  There
you can submit bug reports, make suggestions, and propose pull requests.

### History ###

The ChangeLog of the package is available in
[NEWS.md](https://github.com/giordano/LombScargle.jl/blob/master/NEWS.md) file
in top directory.

License
-------

The `LombScargle.jl` package is licensed under the MIT "Expat" License.  The
original author is Mosè Giordano.
