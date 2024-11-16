---
title: Calculus
slug: /compute-engine/reference/calculus/
---

<Intro>
Calculus is the mathematical study of continuous change. 
</Intro>

It has two main branches: differential calculus and integral calculus. 
These two branches are related by the fundamental theorem of calculus:

$$\int_a^b f(x) \,\mathrm{d}x = F(b) - F(a)$$

where $ F $ is an antiderivative of $ f $, that is $ F' = f $.

**To calculate the derivative of a function**, use the `D` function or `ND`
to calculate a numerical approximation

**To calculate the integral (antiderivative) of a function**, use the `Integrate` function or `NIntegrate` to calculate a numerical approximation.



<FunctionDefinition name="D">

<Signature name="D">_expr_, _var_</Signature>

The `D` function represents the partial derivative of a function `expr` with respect to
the variable `var`.

<Latex value=" f^\prime(x)"/>

```json example
["D", "f", "x"]
```

<Signature name="D">_expr_, _var-1_, _var-2_, ...</Signature>

Multiple variables can be specified to compute the partial derivative of a multivariate
function.

<Latex value=" f^\prime(x, y)"/>

<Latex value=" f'(x, y)"/>

```json example
["D", "f", "x", "y"]
```

<Signature name="D">_expr_, _var_, _var_</Signature>

A variable can be repeated to compute the second derivative of a function.

<Latex value=" f^{\prime\prime}(x)"/>

<Latex value=" f\doubleprime(x)"/>

```json example
["D", "f", "x", "x"]
```

**Explanation**

The derivative is a measure of how a function changes as its input changes. It is the ratio of the change in the value of a function to the change in its input value. 

The derivative of a function $$ f(x) $$ with respect to its input $$ x $$ is denoted by $$ f'(x) $$ or $$ \frac{df}{dx} $$. The derivative of a function $$ f(x) $$ is defined as:

$$
f'(x) = \lim_{h \to 0} \frac{f(x + h) - f(x)}{h}
$$

where:
- $$ h $$ is the change in the input variable.
- $$ f(x + h) - f(x) $$ is the change in the value of the function.
- $$ \frac{f(x + h) - f(x)}{h} $$ is the ratio of the change in the value of the function to the change in its input value.
- $$ \lim_{h \to 0} \frac{f(x + h) - f(x)}{h} $$ is the limit of the ratio of the change in the value of the function to the change in its input value as $$ h $$ approaches $$ 0 $$.
- The limit is taken as $$ h $$ approaches $$ 0 $$ because the derivative is the instantaneous rate of change of the function at a point, and the change in the input value must be infinitesimally small to be instantaneous.

- Wikipedia: [Derivative](https://en.wikipedia.org/wiki/Derivative)
- Wolfram Mathworld: [Derivative](https://mathworld.wolfram.com/Derivative.html)


<b>Reference</b>
- Wikipedia: [Derivative](https://en.wikipedia.org/wiki/Derivative)
- Wikipedia: [Notation for Differentiation](https://en.wikipedia.org/wiki/Notation_for_differentiation), [Leibniz's Notation](https://en.wikipedia.org/wiki/Leibniz%27s_notation), [Lagrange's Notation](https://en.wikipedia.org/wiki/Lagrange%27s_notation),  [Newton's Notation](https://en.wikipedia.org/wiki/Newton%27s_notation)
- Wolfram Mathworld: [Derivative](https://mathworld.wolfram.com/Derivative.html)
- NIST: [Derivative](https://dlmf.nist.gov/2.1#E1)

<b>Lagrange Notation</b>

| LaTeX                 | MathJSON          |
| :-------------------- | :---------------- |
| `f'(x)`               | `["Derivative", "f", "x"]` |
| `f\prime(x)`          | `["Derivative", "f", "x"]` |
| `f^{\prime}(x)`       |   `["Derivative", "f", "x"]` |
| `f''(x)`              | `["Derivative", "f", "x", 2]` |
| `f\prime\prime(x)`    | `["Derivative", "f", "x", 2]` |
| `f^{\prime\prime}(x)` | `["Derivative", "f", "x", 2]` |
| `f\doubleprime(x)` |  `["Derivative", "f", "x", 2]` |
| `f^{(n)}(x)` |  `["Derivative", "f", "x", "n"]` |





</FunctionDefinition>


<FunctionDefinition name="ND">

<Signature name="ND">_expr_, _value_</Signature>

The `ND` function returns a numerical approximation of the partial derivative of a function _expr_ at the point _value_.

<Latex value=" \sin^{\prime}(x)|_{x=1}"/>

```json example
["ND", "Sin", 1]
// ➔ 0.5403023058681398
```

**Note:** `["ND", "Sin", 1]` is equivalent to `["N", ["D", "Sin", 1]]`.


</FunctionDefinition>



<FunctionDefinition name="Derivative">

<Signature name="Derivative">_expr_</Signature>

The `Derivative` function represents the derivative of a function _expr_.

<Latex value=" f^\prime(x)"/>

```json example
["Apply", ["Derivative", "f"], "x"]

// This is equivalent to:
[["Derivative", "f"], "x"]
```



<Signature name="Derivative">_expr_, _n_</Signature>

When a `n` argument is present it represents the _n_-th derivative of a function _expr_.

<Latex value="f^{(n)}(x)"/>

```json example
["Apply", ["Derivative", "f", "n"], "x"]
```


`Derivative` is an operator in the mathematical sense, that is, a function that takes a function
as an argument and returns a function.

The `Derivative` function is used to represent the derivative of a function in a symbolic form. It is not used to calculate the derivative of a function. To calculate the derivative of a function, use the `D` function or `ND` to calculate a numerical approximation.

</FunctionDefinition> 



<FunctionDefinition name="Integrate">
<Signature name="Integrate">_expr_</Signature>

An **indefinite integral**, also known as an antiderivative, refers to the reverse 
process of differentiation. 

<Latex value="\int \sin"/>

```json example
["Integrate", "Sin"]
```

<Signature name="Integrate">_expr_, _index_</Signature>

<Latex value="\int \sin x \,\mathrm{d}x"/>

```json example
["Integrate", ["Sin", "x"], "x"]
```

:::info[Note]
The LaTeX expression above include a LaTeX spacing command `\,` to add a
small space between the function and the differential operator. The differential
operator is wrapped with a `\mathrm{}` command so it can be displayed upright.
Both of these typographical conventions are optional, but they make the 
expression easier to read. The expression `\int \sin x dx` $$\int f(x) dx$$ is equivalent. 
:::


<Signature name="Integrate">_expr_, _bounds_</Signature>

A **definite integral** computes the net area between a function $$ f(x) $$ and
the x-axis over a specified interval $$[a, b]$$. The "net area" accounts for 
areas below the x-axis subtracting from the total. 

<Latex value="\int_{0}^{2} x^2 \,\mathrm{d}x"/>

```json example
["Integrate", ["Power", "x", 2], ["Tuple", "x", 0, 2]]
```

The notation for the definite integral of $$ f(x) $$ from $$ a $$ to $$ b $$ 
is given by:

$$ 
\int_{a}^{b} f(x) \mathrm{d}x = F(b) - F(a) 
$$

where:
- $$ dx $$ indicates the variable of integration.
- $$ [a, b] $$ are the bounds of integration, with $$ a $$ being the lower bound and $$ b $$ being the upper bound.
- $$ F(x) $$ is an antiderivative of $$ f(x) $$, meaning $$ F'(x) = f(x) $$.

Use `NIntegrate` to calculate a numerical approximation of the definite integral of a function.

<Signature name="Integrate">_expr_, _bounds_, _bounds_</Signature>

A **double integral** computes the net volume between a function $$ f(x, y) $$ 
and the xy-plane over a specified region $$[a, b] \times [c, d]$$. The 
"net volume" accounts for volumes below the xy-plane subtracting from the 
total. The notation for the double integral of $$ f(x, y) $$ from $$ a $$ to 
$$ b $$ and $$ c $$ to $$ d $$ is given by:

$$
\int_{a}^{b} \int_{c}^{d} f(x, y) \,\mathrm{d}x \,\mathrm{d}y
$$

<Latex value="\int_{0}^{2} \int_{0}^{3} x^2 \,\mathrm{d}x \,\mathrm{d}y"/>

```json example
["Integrate", ["Power", "x", 2], ["Tuple", "x", 0, 3], ["Tuple", "y", 0, 2]]
```


**Explanation**

Given a function $$f(x)$$, finding its indefinite integral, denoted as 
$$ \int f(x) \,\mathrm{d}x$$, involves finding a new function 
$$F(x)$$ such that $$F'(x) = f(x)$$.

Mathematically, this is expressed as:

$$
\int f(x) \,\mathrm{d}x = F(x) + C 
$$

where:
- $$\mathrm{d}x$$ specifies the variable of integration.
- $$F(x)$$ is the antiderivative or the original function.
- $$C$$ is the constant of integration, accounting for the fact that there are 
  many functions that can have the same derivative, differing only by a constant.

<b>Reference</b>
- Wikipedia: [Integral](https://en.wikipedia.org/wiki/Integral), [Antiderivative](https://en.wikipedia.org/wiki/Antiderivative), [Integral Symbol](https://en.wikipedia.org/wiki/Integral_symbol)
- Wolfram Mathworld: [Integral](https://mathworld.wolfram.com/Integral.html)
- NIST: [Integral](https://dlmf.nist.gov/2.1#E1)


</FunctionDefinition> 

<FunctionDefinition name="NIntegrate">
<Signature name="NIntegrate">_expr_, _lower-bound_, _upper-bound_</Signature>

Calculate the numerical approximation of the definite integral of a function
$$ f(x) $$ from $$ a $$ to $$ b $$.

<Latex value="\int_{0}^{2} x^2 \,\mathrm{d}x"/>

```json example
["NIntegrate", ["Function", ["Power", "x", 2], "x"], 0, 2]
```

</FunctionDefinition>

<FunctionDefinition name="Limit">

<Signature name="Limit">_fn_, _value_</Signature>

Evaluate the expression _fn_ as it approaches the value _value_.

<Latex value=" \lim_{x \to 0} \frac{\sin(x)}{x} = 1"/>


```json example
["Limit", ["Divide", ["Sin", "_"], "_"], 0]

["Limit", ["Function", ["Divide", ["Sin", "x"], "x"], "x"], 0]
```

This function evaluates to a numerical approximation when using `expr.N()`. To
get a numerical evaluation with `expr.evaluate()`, use `NLimit`.



</FunctionDefinition>

<FunctionDefinition name="NLimit">

<Signature name="NLimit">_fn_, _value_</Signature>

Evaluate the expression _fn_ as it approaches the value _value_.

```json example
["NLimit", ["Divide", ["Sin", "_"], "_"], 0]
// ➔ 1

["NLimit", ["Function", ["Divide", ["Sin", "x"], "x"], "x"], 0]
// ➔ 1
```

The numerical approximation is computed using a Richardson extrapolation
algorithm.

</FunctionDefinition>

