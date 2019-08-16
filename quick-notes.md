# Quick intro to calculus with Julia

Here are some `Julia` usages to create calculus objects.

The `Julia` packages loaded below are all loaded when the `CalculusWithJulia` package is loaded.

A `Julia` package is loaded with the `using` command:

```
using LinearAlgebra
```

The `LinearAlgebra` package comes with a `Julia` installation. Other packages can be added. Something like:

```noeval
using Pkg
Pkg.add("SomePackageName")
```

These notes have an accompanying package, `CalculusWithJulia`, that when installed, as above, also installs most of the necessary packages to perform the examples.

Packages need only be installed once, but they must be loaded into *each* session for which they will be used.

```
using CalculusWithJulia
```

Packages can also be loaded through `import PackageName`. Importing does not add the exported objects of a function into the namespace, so is used when there are possible name collisions.

## Types

Objects in `Julia` are "typed." Common numeric types are `Float64`, `Int64` for floating point numbers and integers. Less used here are types like `Rational{Int64}`,  specifying rational numbers with a numerator and denominator as `Int64`; or `Complex{Float64}`, specifying a comlex number with floating point components. Julia also has `BigFloat` and `BigInt` for arbitrary precision types. Typically, operations use "promotion" to ensure the combination of types is appropriate. Other useful types are `Function`, an abstract type describing functions; `Bool` for true and false values; and `Sym` for symbolic values (through `SymPy`).

For the most part the type will not be so important, but it is useful to know that for some function calls the  type of the argument will decide what method ultimately gets called. (This allows symbolic types to interact with Julia functions in an idiomatic manner.)

## Functions

### Definition

Functions can be defined four ways:

* one statement functions follow traditional mathematics notation:

```
f(x) = exp(x) * 2x
```

* multi-statement functions are defined with the `function` keyword. The `end` statement ends the definition. The last evaluated command is returned. There is no need for explicit `return` statement, though it can be useful for control flow.

```
function g(x)
  a = sin(x)^2
  a + a^2 + a^3
end
```


* Anonymous functions are useful for example, as arguments to other functions or as return values.

```
using ForwardDiff   # add-on package to find numeric derivatives using automatic differentiation
D(f::Function) = x -> ForwardDiff.derivative(f, x)
```

The above turns a function that finds the derivative of `f` at a point and creates a function, `D`, returning a derivative function.

* Anonymous function may also be created using the `function` keyword.



For mathematical functions $f: R^n \rightarrow R^m$ when $n$ or $m$ is bigger than 1 we have:

* $n =1$ by $m > 1$ use a "vector" for the return value

```
r(t) = [sin(t), cos(t), t]
```

* $n > 1$ by $m=1$ use multiple arguments

```
f(x,y,z) = x*y + y*z + z*x
f(v) = f(v...)
```

Some functions need to pass in a container of values, for this the last definition is useful to expand the values. Splatting takes a container and treats the values like individual arguments.

Alternatively, indexing can be used directly, as in:

```
f(x) = x[1]*x[2] + x[2]*x[3] + x[3]*x[1]
```

* For vector fields ($n,m > 1$) a combination is used:


```
F(x,y,z) = [-y, x, z]
F(v) = F(v...)
```

### Calling a function

Functions are called using parentheses to group the arguments.

```
f(t) = sin(t)*sqrt(t)
sin(1), sqrt(1), f(1)
```

### Multiple dispatch

`Julia` can have many methods for a single generic function. (E.g., it can have many different implementations of addiion when the `+` sign is encountered.)
The *type* of an argument and the number of arguments are used for dispatch.

Here the number of arguments is used:

```
Area(w, h) = w * h    # area of rectangle
Area(w) = Area(w, w)  # area of square using area of rectangle defintion
```

Calling `Area(5)` will call `Area(5,5)` which will return `5*5`.

Similarly, the definition for a vector field:

```
F(x,y,z) = [-y, x, z]
F(v) = F(v...)
```

takes advantage of multiple dispatch to allow either a vector argument or individual arguments.

Type parameters can be used to restrict the type of arguments that are permitted. The `D(f::Function)` definition illustrates how the `D` function is restricted to `Function` objects.


### Keyword arguments

Optional arguments may be specified with keywords, when the function is defined to use them. Keywords are separated from positional arguments using a semicolon, `;`:

```
circle(x; r=1) = sqrt(r^2 - x^2)
circle(0.5), circle(0.5, r=10)
```





## Symbolic objects

The add-on `SymPy` package allows for symbolic expressions to be used. Symbolic values are defined with `@vars`, as below.

```
using SymPy
@vars x y z  # no comma
x^2 + y^3 + z
```

Assumptions on the variables can be useful, particularly with simplification, as in

```
@vars x y z real=true
```

Symbolic expressions flow through `Julia` functions symbolically

```
sin(x)^2 + cos(x)^2
```

Numbers are symbolic once `SymPy` interacts with them:

```
x - x + 1  # 1 is now symbolic
```

The number `PI` is a symbolic `pi`.
a

```
sin(PI), sin(pi)
```


Use `Sym` to create symbolic numbers, `N` to find a `Julia` number from a symbolic number:

```
1 / Sym(2)
```

```
N(PI)
```

Many `Julia` functions will work with symbolic objects through multiple dispatch (e.g., `sin`, `cos`, ...). Sympy functions that are not in `Julia` can be accessed through the `sympy` object using dot-call notation:

```
sympy.harmonic(10)
```





## Containers

We use a few different containers:

* Tuples. These are objects grouped together using parentheses. They need not be of the same type

```
x1 = (1, "two", 3.0)
```

Tuples are useful for programming. For example, they are uesd to return multiple values from a function.

* Vectors. These are objects of the same type (typically) grouped together using square brackets, values separated by commas:

```
x2 = [1, 2, 3.0]  # 3.0 makes theses all floating point
```

* Matrices. Like vectors, these have the same type, only they are 2-dimensional. Use spaces to separate values along a row; semicolons to separate rows:

```
x3 = [1 2 3; 4 5 6; 7 8 9]
```

* Row vectors. A vector is 1 dimensional, though it may be identified as a column of two dimensional matrix. A row vector is a two-dimensional matrix with a single row:

```
x4 = [1 2 3.0]
```

These have *indexing* using square brackets:

```
x1[1], x2[2], x3[3]
```

Matrices are usually indexed by row and column:

```
x3[1,2] # row one column two
```

For vectors and matrices -- but not tuples -- indexing can be used to change a value in the container:

```
x2[1], x3[1,1] = 2, 2
```

Vectors and matrices are arrays. Arrays have mathematical operations, such as addition and subtraction, defined for them. Tuples do not.


Destructuring is another way to get at the entries:

```
a,b,c = x2
```

### structured collections

An arithmetic progression, $a, a+h, a+2h, ..., b$ can be produced *efficiently* using the range operator:

```
5:10:55  # an object that describes 5, 15, 25, 35, 45, 55
```

If `h=1` it can be omitted:

```
1:10     # an object that describes 1,2,3,4,5,6,7,8,9,10
```

The `range` function can *efficiently* describe $n$ evenly spaced points between `a` and `b`:

```
range(0, pi, length=5)  # range(a, stop=b, length=n) for version 1.0
```

This is useful for creating regularly spaced values needed for certain plots.

## Iteration


The `for` keyword is useful for iteration, Here is a traditional for loop, as `i` loops over each entry of the vector `[1,2,3]`:

```
for i in [1,2,3]
  print(i)
end
```

List comprehensions are similar, but are useful as they perform the iteration and collect the values:

```
[i^2 for i in [1,2,3]]
```

Comprehesions can also be used to make matrices

```
[1/(i+j) for i in 1:3, j in 1:4]
```


Comprehensions  apply an *expression* to each entry in a container through iteration. Applying a function to each entry of a container can be facilitated by:

* Broadcasting. Using  `.` before an operation instructs `Julia` to match up sizes (possibly extending to do so) and then apply the operation element by element:

```
xs = [1,2,3]
sin.(xs)   # sin(1), sin(2), sin(3)
bases = [5,5,10]
log.(bases, xs)  # log(5, 1), log(5,2), log(10, 3)
```

Row and column vectors can fill in:

```
ys = [4 5] # a row vector
f(x,y) = (x,y)
f.(xs, ys)    # broadcasting a column and row vector makes a matrix, then applies f.
```

* The `map` function applies a function to each element:

```
map(sin, [1,2,3])
```

## Plots


The following commands use the `Plots` package. The `Plots` package expects a choice of backend. We will use both `plotly` and `gr` (and occasionally `pyplot()`).

```
using Plots
plotly()      # select plotly. Use `gr()` for GR
```

> Plotting a univariate function $f:R \rightarrow R$

* using `plot(f, a, b)`

```
plot(sin, 0, 2pi)
```

Or

```
f(x) = exp(-x/2pi)*sin(x)
plot(f, 0, 2pi)
```

Or with an anonymous function

```
plot(x -> sin(x) + sin(2x), 0, 2pi)
```

```
note("""The time to first plot can be lengthy!. This can be removed by creating a custom `Julia` image, but that is no introductory level stuff. As well, standalone plotting packages offer quicker first plots, but the simplicity of `Plots` is preferred. Subsequent plots are not so time consuming, as the initial time is spent compiling functions so their re-use is speedy.
""")
```


Arguments of interest include

| Attribute      | Value                                                   |
|:--------------:|:-------------------------------------------------------:|
| `legend`       | A boolean, specify `false` to inhibit drawing a legend  |
| `aspect_ratio` | Use `:equal` to have x and y axis have same scale       |
| `linewidth`    | Ingters greater than 1 will thicken lines drawn         |
| `color`        | A color may be specified by a symbol (leading `:`).     |
|                | E.g., `:black`, `:red`, `:blue`                          |



* using `plot(xs, ys)`

The lower level interface to plot involves creating x and y values to plot:

```
xs = range(0, 2pi, length=100)
ys = sin.(xs)
plot(xs, ys, color=:red)
```


* plotting a symbolic expression

A symbolic expression of single variable can be plotted as a function is:

```
@vars x
plot(exp(-x/2pi)*sin(x), 0, 2pi)
```

* Multiple functions

The `!` Julia convention to modify an object is used by the `plot` command, so `plot!` will add to the existing plot:

```
plot(sin, 0, 2pi, color=:red)
plot!(cos, 0, 2pi, color=:blue)
plot!(zero, color=:green)  # no a, b then inherited from graph.
```

The `zero` function is just 0 (useful when the type of a number is important).

> Plotting a parameterized (space) curve function $f:R \rightarrow R^n$, $n = 2$ or $3$

* Using `plot(xs, ys)`

Let $f(t) = e^{t/2\pi} \langle \cos(t), \sin(t)\rangle$ be a parameterized function. Then the $t$ values can be generated as follows:

```
ts = range(0, 2pi, length = 100)
xs = [exp(t/2pi) * cos(t) for t in ts]
ys = [exp(t/2pi) * sin(t) for t in ts]
plot(xs, ys)
```

* using `plot(f1, f2, a, b)`. If the two functions describing the components are available, then

```
f1(t) = exp(t/2pi) * cos(t)
f2(t) = exp(t/2pi) * sin(t)
plot(f1, f2, 0, 2pi)
```

* Using `plot_parametric_curve`. If the curve is described as a function of `t` with a vector output, then the `CalculusWithJulia` package provides `plot_parametric_curve` to produce a plot:

```
r(t) = exp(t/2pi) * [cos(t), sin(t)]
plot_parametric_curve(r, 0, 2pi)
```

The low-level approach doesn't quite work as easily as desired:

```
ts = range(0, 2pi, length = 4)
vs = r.(ts)
```

As seen, the values are a vector of vectors. To plot a reshaping needs to be done:

```
ts = range(0, 2pi, length = 100)
vs = r.(ts)
xs = [vs[i][1] for i in eachindex(vs)]
ys = [vs[i][2] for i in eachindex(vs)]
plot(xs, ys)
```

This approach is faciliated by the `unzip` function in `CalculusWithJulia`:

```
plot(unzip(vs)...)
```



* Plotting an arrow

An arrow in 2D can be plotted with the `quiver` command. We show the `arrow(p, v)` (or `arrow!(p,v)` function) from the `CalculusWithJulia` package, which has an easier syntax:

```
gr()
plot_parametric_curve(r, 0, 2pi)
t0 = pi/8
arrow!(r(t0), r'(t0))
```

The `GR` package makes nicer arrows that `Plotly`.

> Plotting a scalar function $f:R^2 \rightarrow R$

The `surface` and `contour` functions are available to visualize a scalar function of $2$ variables:

* A surface plot

(The `plotly` backend allows for rotation by the mouse.)

```
plotly()
f(x, y) = 2 - x^2 + y^2
xs = ys = range(-2,2, length=25)
surface(xs, ys, f)
```

The function generates the $z$ values, this can be done by the user and then passed to the `surface(xs, ys, zs)` format:

```
surface(xs, ys, f.(xs', ys))
```





* A contour plot

The `contour` function is like the `surface` function.

```
contour(xs, ys, f)
```

```
contour(xs, ys, f.(xs', ys))
```


* An implicit equation. The constraint $f(x,y)=c$ generates an implicit equation. While contour can be used for this plot by adjusting the requested contours, the `ImplicitEquations` package can as well, and perhaps is easier. This package is loaded with `CalculusWithJulia`; loading it by itself will lead to naming conflicts with `SymPy`, so best not to do so. `ImplicitEquations` plots predicates formed by `Eq`, `Le`, `Lt`, `Ge`, and `Gt` (or some unicode counterparts). For example to plot when $f(x,y) =  \sin(xy) - \cos(xy) \leq 0$ we have:

```
f(x,y) = sin(x*y) - cos(x*y)
plot(Le(f, 0))
```


> Plotting a parameterized surface $f:R^2 \rightarrow R^3$

The `plotly` (and `pyplot`) backends allow plotting of parameterized surfaces. The `plot_parametric_surface` function from `CalculusWithJulia` handles the details:

```
plotly()
Phi(theta, phi) = [sin(phi) * cos(theta), sin(phi) * sin(theta), cos(phi)]
plot_parametric_surface(Phi, xlim=(0,pi/4), ylim=(0,pi))
```


The low-level `surface(xs,ys,zs)` is used, and can be specified directly as follows:

```
X(theta, phi) = sin(phi)*cos(theta)
Y(theta, phi) = sin(phi)*sin(theta)
Z(theta, phi) = cos(phi)
thetas = range(0, pi/4, length=20)
phis = range(0, pi, length=20)
surface(X.(thetas, phis'), Y.(thetas, phis'), Z.(thetas, phis'))
```

> Plotting a vector field $F:R^2 \rightarrow R^2$. The `CalculusWithJulia` package provides `vectorfieldplot`, used as:

```
gr()  # better arrows
F(x,y) = [-y, x]
vectorfieldplot(F, xlim=(-2, 2), ylim=(-2,2), nx=10, ny=10)
```

There is also `vectorfieldplot3d`.


## Limits

Limits can be investigated numerically by forming tables, eg.:

```
xs = [1, 1/10, 1/100, 1/1000]
f(x) = sin(x)/x
[xs f.(xs)]
```

Symbolically, `SymPy` provides a `limit` function:

```
@vars x
limit(sin(x)/x, x => 0)
```

Or

```
@vars h x
limit((sin(x+h) - sin(x))/h, h => 0)
```

## Derivatives

There are numeric and symbolic approaches  to derivatives. For the numeric approach we use the `ForwardDiff` package, which performs automatic differentiation.


> Derivatives of univariate functions

Numerically, the `ForwardDiff.derivative(f, x)` function call will find the derivative of the function `f` at the point `x`:

```
ForwardDiff.derivative(sin, pi/3) - cos(pi/3)
```

The `CalculusWithJulia` package overides the `'` (`adjoint`) syntax for functions to provide a derivative which takes a function and returns a function, so its usage is familiar

```
f(x) = sin(x)
f'(pi/3) - cos(pi/3)  # or just sin'(pi/3) - cos(pi/3)
```

Higher order derivatives are possible as well,


```
f(x) = sin(x)
f''''(pi/3) - f(pi/3)
```


----

Symbolically, the `diff` function of `SymPy` finds derivatives.

```
@vars x
f(x) = exp(-x)*sin(x)
ex = f(x)  # symbolic expression
diff(ex, x)   # or just diff(f(x), x)
```

Higher order derivatives can be specified as well

```
diff(ex, x, x)
```

Or with a number:

```
diff(ex, x, 5)
```

The variable is important, as this allows parameters to be symbolic

```
@vars mu sigma x
diff(exp(-((x-mu)/sigma)^2/2), x)
```

> partial derivatives

There is no direct partial derivative function provided by `ForwardDiff`, rather we use the result of the `ForwardDiff.gradient` function, which finds the partial derivatives for each variable. To use this, the function must be defined in terms of a point or vector.

```
f(x,y,z) = x*y + y*z + z*x
f(v) = f(v...)               # this is needed for ForwardDiff.gradient
ForwardDiff.gradient(f, [1,2,3])
```

We can see directly that $\partial{f}/\partial{x} = \langle y + z\rangle$. At the point $(1,2,3)$, this is $5$, as returned above.

----

Symbolically, `diff` is used for partial derivatives:

```
@vars x y z
ex = x*y + y*z + z*x
diff(ex, x)    # ∂f/∂x
```

> Gradient

As seen, the `ForwardDiff.gradient` function finds the gradient at a point. In `CalculusWithJulia`, the gradient is extended to returns a function when called with no additional arguments:

```
f(x,y,z) =  x*y + y*z + z*x
f(v) = f(v...)
gradient(f)(1,2,3) - gradient(f, [1,2,3])
```

The `∇` symbol, formed by entering `\nabla[tab]`, is mathematical syntax for the gradient, and is defined in `CalculusWithJulia`.

```
∇(f)(1,2,3)   # same as gradient(f, [1,2,3])
```

----

In `SymPy`, there is no gradient function, though finding the gradient is easy through broadcasting:

```
@vars x y z
ex = x*y + y*z + z*x
diff.(ex, [x,y,z])  # [diff(ex, x), diff(ex, y), diff(ex, z)]
```

The  `CalculusWithJulia` package provides a method for `gradient`:

```
gradient(ex, [x,y,z])
```

The `∇` symbol is an alias. It can guess the order of the free symbols, but generally specifying them is needed. This is done with a tuple:

```
∇((ex, [x,y,z]))
```


> Jacobian

The Jacobian of a function $f:R^n \rightarrow R^m$ is a $m\times n$ matrix of partial derivatives. Numerically, `ForwardDiff.jacobian` can find the Jacobian of a function at a point:

```
F(u,v) = [u*cos(v), u*sin(v), u]
F(v) = F(v...)    # needed for ForwardDiff.jacobian
pt = [1, pi/4]
ForwardDiff.jacobian(F , pt)
```

----

Symbolically, the `jacobian` function is a method of a *matrix*, so the calling pattern is different. (Of the form `object.method(arguments...)`.)

```
@vars u v
ex = F(u,v)
ex.jacobian([u,v])
```

As the Jacobian can be identified as the matrix with rows given by the transpose of the gradient of the component, it can be computed directly, but it is more difficult:

```
@vars u v real=true
vcat([diff.(ex, [u,v])' for ex in F(u,v)]...)
```

> Divergence

Numerically, the divergence can be computed from the Jacobian by adding the diagonal elements. This is a numerically inefficient, as the other partial derivates must be found and discarded, but this is generally not an issue for these notes. The following uses `tr` (the trace from the `LinearAlgebra` package) to find the sum of a diagonal.

```
F(x,y,z) = [-y, x, z]
F(v) = F(v...)
pt = [1,2,3]
tr(ForwardDiff.jacobian(F , pt))
```

The `CalculusWithJulia` package provides `divergence` to compute the divergence and provides the `∇ ⋅` notation (`\nabla[tab]\cdot[tab]`):

```
divergence(F, [1,2,3])
(∇⋅F)(1,2,3)    # not ∇⋅F(1,2,3) as that evaluates F(1,2,3) before the divergence
```

----


Symbolically, the divergence can be found directly:

```
@vars x y z
ex = F(x,y,z)
sum(diff.(ex, [x,y,z]))    # sum of [diff(ex[1], x), diff(ex[2],y), diff(ex[3], z)]
```

The `divergence` function can be used for symbolic expressions:

```
divergence(ex, [x,y,z])
∇⋅(F(x,y,z), [x,y,z])
```

> Curl

The curl can be computed from the off-diagonal elements of the Jacobian. The calculation follows the formula. The `CalculusWithJulia` package provides `curl` to compute this:

```
F(x,y,z) = [-y, x, 1]
F(v) = F(v...)
curl(F, [1,2,3])
```

As well, if no point is specified, a function is returned for which a point may be specified using 3 coordinates or a vector

```
curl(F)(1,2,3), curl(F)([1,2,3])
```

Finally, the `∇ ×` (`\nabla[tab]\times[tab]` notation is available)
```
(∇×F)(1,2,3)
```

----
## Integrals

Numeric integration is provided by the `QuadGK` package, for univariate integrals, and the `HCubature` package for higher dimensional integrals.

> Integrals of univariate functions

A definite integral may be computed numerically using `quadgk`

```
using QuadGK
quadgk(sin, 0, pi)
```

The answer and an estimate for the worst case error is returned.

If singularities are avoided, improper integrals are computed as well:

```
quadgk(x->1/x^(1/2), 0, 1)
```


-----

SymPy provides the `integrate` function to compute both definite and indefinite integrals.


```
@vars a x real=true
integrate(exp(a*x)*sin(x), x)
```

Like `diff` the variable to integrate is specified.

Definite integrals use a tuple, `(variable, a, b)` to integrate:

```
integrate(sin(a + x), (x, 0, PI))   # \int_0^PI sin(a+x) dx
```




> 2D and 3D iterated integrals

Two and three dimensional integrals over box-like regions are computed numerically with the `hcubature` function from the `HCubature` package. If the box is $[x_1, y_1]\times[x_2,y_2]\times\cdots\times[x_n,y_n]$ then the limits are specified through tuples of the form $(x_1,x_2,\dots,x_n)$ and $(y_1,y_2,\dots,y_n)$.

```
f(x,y) = x*y^2
f(v) = f(v...)
hcubature(f, (0,0), (1, 2))  # computes \int_0^1\int0^2 f(x,y) dy dx
```

The calling pattern for more dimensions is identical.

```
f(x,y,z) = x*y^2*z^3
f(v) = f(v...)
hcubature(f, (0,0,0), (1, 2,3))
```

The box-like region requirement means a change of variables may be necessary. For example, to integrate over the region $x^2 + y^2 \leq 1; x \geq 0$, polar coordinates can be used with $(r,\theta)$ in $[0,1]\times[-\pi/2,\pi/2]$. When changing variables, the Jacobian enters into the formula, through

$$~
\iint_{G(S)} f(\vec{x}) dV = \iint_S (f \circ G)(\vec{u}) |\det(J_G)(\vec{u})| dU.
~$$

Here we implement this:

```
f(x,y) = x*y^2
f(v) = f(v...)
Phi(r, theta) = r * [cos(theta), sin(theta)]
Phi(rtheta) = Phi(rtheta...)
integrand(rtheta) = f(Phi(rtheta)) * det(ForwardDiff.jacobian(Phi, rtheta))
hcubature(integrand, (0.0,-pi/2), (1.0, pi/2))
```

In `CalculusWithJulia` a `fubini` function is provided to compute numeric integrals over regions which can be described by curves represented by functions. E.g., for this problem:

```
fubini(f, (x -> -sqrt(1-x^2), x -> sqrt(1-x^2)), (0, 1))
```

This function is for convenience, but is not performant.


----

Symbolically, the `integrate` function allows additional terms to be specified. For example, the above could be done through:

```
@vars x y real=true
integrate(x * y^2, (y, -sqrt(1-x^2), sqrt(1-x^2)), (x, 0, 1))
```


> Line integrals

A line integral of $f$ parameterized by $\vec{r}(t)$ is computed by:

$$~
\int_a^b (f\circ\vec{r})(t) \| \frac{dr}{dt}\| dt.
~$$

For example, if $f(x,y) = 2 - x^2 - y^2$ and $r(t) = 1/t \langle \cos(t), \sin(t) \rangle$, then the line integral over $[1,2]$ is given by:

```
f(x,y) = 2 - x^2 - y^2
f(v) = f(v...)
r(t) = [cos(t), sin(t)]/t
integrand(t) = (f∘r)(t) * norm(r'(t))
quadgk(integrand, 1, 2)
```

To integrate a line integral through a vector field, say $\int_C F \cdot\hat{T} ds=\int_C F\cdot \vec{r}'(t) dt$ we have, for example,

```
F(x,y) = [-y, x]
F(v) = F(v...)
r(t) =  [cos(t), sin(t)]/t
integrand(t) = (F∘r)(t) ⋅ r'(t)
quadgk(integrand, 1, 2)
```

----

Symbolically, there is no real difference from a 1-dimensional integral. Let $\phi = 1/\|r\|$ and integrate the gradient field over one turn of the helix $\vec{r}(t) = \langle \cos(t), \sin(t), t\rangle$.

```
@vars x y z t real=true
phi(x,y,z) = 1/sqrt(x^2 + y^2 + z^2)
r(t) = [cos(t), sin(t), t]
∇phi = diff.(phi(x,y,z), [x,y,z])
∇phi_r = subs.(∇phi, x.=> r(t)[1], y.=>r(t)[2], z.=>r(t)[3])
rp = diff.(r(t), t)
ex = simplify(∇phi_r ⋅ rp )
```

Then

```
integrate(ex, (t, 0, 2PI))
```

> surface integrals


The surface integral for a parameterized surface involves a surface element $\|\partial\Phi/\partial{u} \times \partial\Phi/\partial{v}\|$. This can be computed numerically with:

```
Phi(u,v) = [u*cos(v), u*sin(v), u]
Phi(v) = Phi(v...)
function SE(Phi, pt)
   J = ForwardDiff.jacobian(Phi, pt)
   J[:,1] × J[:,2]
end
norm(SE(Phi, [1,2]))
```

To find the surface integral ($f=1$) for this surface over $[0,1] \times [0,2pi]$, we have:

```
hcubature(pt -> norm(SE(Phi, pt)), (0.0,0.0), (1.0, 2pi))
```

Symbolically, the approach is similar:

```
@vars u v real=true
ex = Phi(u,v)
J = ex.jacobian([u,v])
SurfEl = norm(J[:,1] × J[:,2]) |> simplify
```

Then

```
integrate(SurfEl, (u, 0, 1), (v, 0, 2PI))
```