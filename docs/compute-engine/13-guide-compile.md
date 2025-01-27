---
title: Compiling Expressions
slug: /compute-engine/guides/compiling/
---

<Intro>
The Compute Engine **LaTeX expressions** can compile expressions to **JavaScript functions**!
</Intro>

## Introduction

Some expressions can take a long time to evaluate numerically, for example if
they contain a large number of terms or involve a loop \\((\sum\\) or
\\(\prod\\)).

In this case, it is useful to compile the expression into a JavaScript function
that can be evaluated much faster.

For example this approximation of \\(\pi\\): \\(
\sqrt\{6\sum^\{10^6\}\_\{n=1\}\frac\{1\}\{n^2\}\} \\)

```javascript
const expr = ce.parse("\\sqrt{6\\sum^{10^6}_{n=1}\\frac{1}{n^2}}");

// Numerical evaluation using the Compute Engine
console.log(expr.evaluate().latex);
// ➔ 3.14159169866146
// Timing: 1,531ms

// Compilation to a JavaScript function and execution
const fn = expr.compile();
console.log(fn());
// ➔ 3.1415916986605086
// Timing: 6.2ms (247x faster)
```

## Compiling

**To get a compiled version of an expression** use the `expr.compile()` method:

```javascript
const expr = ce.parse("2\\prod_{n=1}^{\\infty} \\frac{4n^2}{4n^2-1}");
const fn = expr.compile();
```

**To evaluate the compiled expression** call the function returned by
`expr.compile()`:

```javascript
console.log(fn());
// ➔ 3.141592653589793
```

If the expression cannot be compiled, the `compile()` method will return
`undefined`.

## Arguments

The function returned by `expr.compile()` can be called with an object literal
containing the value of the arguments:

```javascript
const expr = ce.parse("n^2");
const fn = expr.compile();
for (const i = 1; i < 10; i++) console.log(fn({ n: i }));
// ➔ 1, 4, 9, 16, 25, 36, 49, 64, 81
```

**To get a list of the unknows of an expression** use the `expr.unknowns`
property:

```javascript
console.log(ce.parse("n^2").unknowns);
// ➔ ["n"]

console.log(ce.parse("a^2+b^3").unknowns);
// ➔ ["a", "b"]
```

## Limitations

The calculations are only performed using machine precision numbers.

Complex numbers, arbitrary precision numbers, and symbolic calculations are not
supported.

Some functions are not supported.

If the expression cannot be compiled, the `compile()` method will return
`undefined`. The expression can be numerically evaluated as a fallback:

```live
const expr = ce.parse("-i\\sqrt{-1}");
console.log(expr.compile() ?? expr.N().numericValue);
// Compile cannot handle complex numbers, so it returns `undefined`
// and we fall back to numerical evaluation with expr.N()
// ➔ 1
```
