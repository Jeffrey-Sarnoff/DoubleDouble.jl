TwoFloat.jl
===========

`TwoFloat.jl` is a Julia package for performing extended-precision arithmetic using pairs of floating-point numbers, based on the ideas and algorithms of [Dekker (1971)][dekker1971]. Its aim is to provide accurate results without the overhead of `BigFloat` types.

Interface
------------
The two basic types are `Double` and `Split`; both have two floating-point fields: `hi`, storing the leading bits, and `lo` storing the remainder. 

A `Double` is the simplest type, with `hi` stored to full precision, rounded to nearest. As a result for any `Double` `x`, `x.lo <= 0.5*eps(x.hi)`.

```julia
julia> using TwoFloat

julia> x = double(pi)
Double{Float64}(3.141592653589793,1.2246467991473532e-16)

julia> eps(x.hi)
4.440892098500626e-16
```

In a `Split`, the `hi` field is stored only to half-precision: e.g., a `Float64` has a 53 bit mantissa, so only the first 26 bits are used. 
```julia
julia> s = split(pi)
Split{Float64}(3.1415926814079285,-2.7818135228334233e-8)

julia> bits(s.hi)
"0100000000001001001000011111101101011000000000000000000000000000"
```
The advantage of this is that the product of two half-precision floats can be stored exactly in a full precision float.

Example: exact product and remainder
-------------------------------------
By exploiting this property, we can compute exact products of floating point numbers.

```julia
julia> u, v = 64*rand(), 64*rand()
(15.59263373822506,39.07676672446341)

julia> w = split(u)*split(v)
Double{Float64}(609.3097112086186,-5.3107663829696295e-14)
```
Note that the product of two `Split`s is a `Double`: the `hi` element of this double is equal to the usual product. If both `Split`s are derived from exact floats (and not other numbers, such as `pi`), this product will be exact.

This can be used to get an accurate remainder 
```julia
julia> r = rem(w,1.0)
Double{Float64}(0.309711208618584,1.6507898617445858e-17)

julia> float64(r)
0.309711208618584
```

This is much more accurate than taking ordinary products, and gives the same answer as using `BigFloat`s:
```julia
julia> rem(u*v,1.0)
0.3097112086186371

julia> float64(rem(big(u)*big(v),1.0))
0.309711208618584
```
However the `TwoFloat` version is all performed using ordinary floating point operations, and is approximately 1000x faster than the `BigFloat` version.

Example: correct rounding with non-exact floats
------------------------------------------------
If a number cannot be exactly represented by a floating point number, it may be rounded incorrectly when used later, e.g.
```julia
julia> pi*0.1
0.3141592653589793

julia> float64(big(pi)*0.1)
0.31415926535897937
```
We can also do this computation using `Split`s (note that the promotion rules mean that only one needs to be a `Split`):
```julia
julia> float64(split(pi)*0.1)
0.31415926535897937

julia> float64(pi*split(0.1))
0.31415926535897937
```

[dekker1971]: http://link.springer.com/article/10.1007%2FBF01397083  "T.J. Dekker (1971) 'A floating-point technique for extending the available precision', Numerische Mathematik, Volume 18, Issue 3, pp 224-242"