---
title: Expressions
slug: /compute-engine/guides/expressions/
layout: single
date: Last Modified
sidebar:
  - nav: "universal"
toc: true
---

<Intro>
The Compute Engine produces and manipulates
[symbolic expressions](<https://en.wikipedia.org/wiki/Expression_(mathematics)>)
such as numbers, constants, variables and functions.
</Intro>

In the Compute Engine, expressions are represented internally using the
[MathJSON format](/math-json/).

They are wrapped in a JavaScript object, a process called **boxing**, and the
resulting expressions are **Boxed Expressions**.

Boxed Expressions improve performance by implementing caching to avoid
repetitive calculations. They also ensure that expressions are valid and in a
standard format.

Unlike the plain data types used by JSON, Boxed Expressions allow an IDE, such
as VSCode Studio, to provide suitable hints in the editor regarding which
methods and properties are available for a given expression.

Boxed Expression can be created from a LaTeX string or from a raw MathJSON
expression.

## Boxing

**To create a Boxed Expression from a MathJSON expression**, use the `ce.box()`
function.

The input of `ce.box()` can be:
- a [MathJSON expression](/math-json/)
- a `BoxedExpression` (in which case it is returned as-is)
- a `SemiBoxedExpression`, that is a MathJSON expression with some of its
  subexpressions already boxed.

The result is an instance of a `BoxedExpression`.

By default, `ce.box()` returns a canonical expression. See
[Canonical Expressions](#canonical) for more info.


```js
let expr = ce.box(1.729e3);
console.log(expr.re);
// ➔ 1729

console.log(expr.isPositive);
// ➔ true

expr = ce.box({ num: "+Infinity" });
console.log(expr.latex);
// ➔ "+\infty"

expr = ce.box(["Add", 3, "x"]);
console.log(expr.operator);
// ➔ "Add"
```


**To create a Boxed Expression from a LaTeX string**, call the `ce.parse()`
function.

```js
const expr = ce.parse("3 + x + y");
console.log(expr.operator);
// ➔ "Add"

console.log(expr.json);
// ➔ ["Add", 3, "x", "y"]
```

By default, `ce.parse()` returns a canonical expression. See
[Canonical Expressions](#canonical-expressions) for more info.

**To get a Boxed Expression representing the content of a mathfield**
use the `mf.expression` property:

```js
const mf = document.getElementById("input");
mf.value = "\\frac{10}{5}";
const expr = mf.expression;
console.log(expr.evaluate().latex);
// ➔ 2
```

## Unboxing

**To access the MathJSON expression of a boxed expression as plain JSON**, use
the `expr.json` property. This property is an "unboxed" version of the
expression.

```js
const expr = ce.box(["Add", 3, "x"]);
console.log(expr.json);
// ➔ ["Add", 3, "x"]
```

**To customize the format of the MathJSON expression returned by `expr.json`**
use the `ce.toMathJson` method.

Use this option to control:

- which metadata, if any, should be included
- whether to use shorthand notation
- to exclude some functions.

See [JsonSerializationOptions](/docs/compute-engine/?q=JsonSerializationOptions)
for more info about the formatting options available.

```ts
const expr = ce.parse("2 + \\frac{q}{p}");
console.log(expr.json);
// ➔ ["Add", 2, ["Divide", "q", "p"]]

console.log(expr.toMathJson({
  exclude: ["Divide"], // Don't use `Divide` functions,
  // use `Multiply`/`Power` instead
  shorthands: [], // Don't use any shorthands
}));
// ➔ ["fn": ["Add", ["num": "2"],
//      ["fn": ["Multiply",
//        ["sym": "q"],
//        ["fn": ["Power", ["sym": "p"], ["num": "-1"]]]]
//      ]
//    ]]
```

<section id="canonical">

## Canonical Expressions

The **canonical form** of an expression is a conventional way of writing an
expression.

For example, the canonical form of a fraction of two integers is a reduced
rational number, written as a tuple of two integers, such that the GCD of the
numerator and denominator is 1, and the denominator is positive.

```js
const expr = ce.parse("\\frac{30}{-50}");
console.log(expr.json);
// ➔ ["Rational", -3, 5]
```

The canonical form of a rational with a denominator of 1 is an integer.

```js
const expr = ce.parse("\\frac{17}{1}");
console.log(expr.json);
// ➔ 17
```

You can check if a non-canonical expression is a reduced (canonical) rational 
number by checking the GCD of the numerator and denominator:

```js example
const input = ce.parse("\\frac{30}{50}", {canonical: false});
console.info(ce.box(
  ["GCD", ["NumeratorDenominator", input]]
).evaluate().value === 1);
// ➔ false
```


The canonical form of an addition or multiplication will have its arguments
ordered in a canonical way.

```js
const expr = ce.parse("2+x+\\pi+\\sqrt2+1");
console.log(expr.json);
// ➔ ["Add", "Pi", ["Sqrt", 2], "x", 1, 2]
```

<ReadMore path="/compute-engine/guides/canonical-form/" > 
Read more about the **Canonical Form** <Icon name="chevron-right-bold" />
</ReadMore>

By default, `ce.box()` and `ce.parse()` produce a canonical expression.

**To get a non-canonical expression instead**, use
`ce.box(expr, {canonical: false})` or `ce.parse(latex, {canonical: false})`.

The non-canonical form sticks closer to the original LaTeX input.

```js
const expr = "\\frac{30}{-50}";

ce.parse(expr);
// canonical form ➔ ["Rational", -3, 5]

ce.parse(expr, { canonical: false });
// non-canonical form ➔ ["Divide", 30, -50]
```

**To obtain the canonical representation of a non-canonical expression**, use
`expr.canonical`.

A non-canonical expression may include errors as a result of parsing from LaTeX,
if the LaTeX input contained LaTeX syntax errors.

A canonical expression may include additional errors compared to a non-canonical
expression, for example `["Divide", 2, 5, 6]` (three arguments instead of two),
`["Add", 2, "True"]` (mismatched argument type, expected a number but got a
boolean).

The canonical form of an expression which is not valid will include one or more
`["Error"]` expressions indicating the nature of the problem.

**To check if an expression contains errors** use `expr.isValid`.

When doing this check on a canonical expression it takes into consideration not
only possible syntax errors, but also semantic errors (incorrect number or
type of arguments, etc...).

</section>

## Mutability

Unless otherwise specified, expressions are immutable.

The functions that manipulate Boxed Expressions, such as `expr.simplify()`,
`expr.evaluate()`, `expr.N()` return a new Boxed Expression, without modifying
`expr`.

However, the properties of the expression may change, since some of them may
depend on contextual information which can change over time.

For example, `ce.box('n').isPositive` may return `undefined` if nothing is known 
about the symbol `n`. But if an assumption about the symbol is made later, or a value
assigned to it, then `ce.box('n').isPositive` may take a different value.

```js
const expr = ce.box("n");
console.log(expr.isPositive);
// ➔ undefined

ce.assume(ce.parse("n > 0"));
console.log(expr.isPositive);
// ➔ true
```

What doesn't change is the fact that `expr` represents the symbol `"n"`.

## Pure Expressions

A pure expression is an expression whose value is fixed. Evaluating it produces
no side effect.

The \\( \sin() \\) function is pure: it evaluates to the same value when the
same arguments are applied to it.

On the other hand, the \\( \operatorname\{window\}() \\) function is not pure: by
its nature it evaluates to a different value on every evaluation.

Numbers, symbols and strings are pure. A function expression is pure if the
function itself is pure, and all its arguments are pure as well.

**To check if an expression is pure**, use `expr.isPure`.

## Checking the Kind of Expression

To identify if an expression is a number, symbol, function or string
use the following boolean expressions:

<div className="symbols-table first-column-header">

| Kind           | Boolean Expression                                     |
| :------------- | :----------------------------------------------------- |
| **Number**     | `expr.isNumberLiteral`                           |
| **Symbol**     | `expr.symbol !== null` |
| **Function**   | `expr.ops !== null`                                    |
| **String**     | `expr.string !== null`  |

</div>

The value of `expr.numericValue` may be:

- `typeof expr.numericValue === "number"`: the expression is a JavaScript number
- otherwise, the property is a `NumericValue` object.

**To access a the value of an expression as a JavaScript primitive**, use
`expr.value`. The result is a JavaScript primitive, such as a number, string or
boolean.

**To access the value of an expression as a JavaScript number**, use
`expr.re`. The result is the real part of the number, as a JavaScript number, 
or `NaN` if the expression is not a number. Use `expr.im` to get the imaginary part.


## Errors

Sometimes, things go wrong.

When something goes wrong the Compute Engine uses an
`["Error", <cause>, <location>]` expression.

The `<cause>` argument provides details about the nature of the problem. This
can be either a string or an `["ErrorCode"]` expression if there are additional
arguments to the error.

For example if the problem is that an argument of a function expression is a
boolean when a number was expected, an expression such as
`["Error", ["ErrorCode", "'incompatible-type'", "'number'", "'boolean'"]]` could
be returned.

The `<location>` argument indicates the context of the error. This can be a
`["Latex"]` expression when the problem occurred while parsing a LaTeX string,
or another expression if the problem was detected later.

### Parsing Errors

When parsing a LaTeX expression, the Compute Engine uses the **maximum effort**
doctrine. That is, even partially complete expressions are parsed, and as much
of the input as possible is reflected in the MathJSON result.

If required operands are missing (the denominator of a fraction, for example), a
`["Error", ""missing""]` error expression is inserted where the missing operand
should have been.

Problems that occur while parsing a LaTeX string will usually indicate a LaTeX
syntax error or typo: missing `}`, mistyped command name, etc...

### Semantic Errors

Some errors are not caught until an expression is bound, that is until an
attempt is made to associate its symbol or function identifiers to a definition.
This could include errors such as missing or mismatched arguments.

Some errors that could be considered LaTeX syntax errors may not surface until
binding occurs.

For example `\frac{1}{2=x}` (instead of `\frac{1}{2}=x`) will be parsed as
`["Divide", 1, ["Equal", 2, x]]`. The fact that the second argument of the
`"Divide"` function is a boolean and not a number will not be detected until the
definition for `"Divide"` has been located.

Name binding is done lazily, not upon boxing. To force the binding to occur,
request the canonical version of the expression.

**To check if an expression includes an `["Error"]` subexpression** check the
`expr.isValid` property.

**To get the list of all the `["Error"]` subexpression** use the `expr.errors`
property.

<div className="symbols-table first-column-header">

| Error Code                     | Meaning                                                                                                          |
| :----------------------------- | :--------------------------------------------------------------------------------------------------------------- |
| `syntax-error`                 | the parsing could not continue                                                                                   |
| `missing`                      | an expression was expected                                                                                       |
| `expected-expression`          | an expression was expected inside an enclosure (parentheses)                                                     |
| `unexpected-command`           | the command is unknown, or not applicable in the current parsing context                                         |
| `unexpected-token`             | the character does not apply to the current parsing context                                                      |
| `incompatible-domain`          | the argument provided does not match the expected domain                                                         |
| `unexpected-argument`          | too many arguments provided                                                                                      |
| `expected-argument`            | not enough arguments provided                                                                                    |
| `invalid-identifier`           | the identifier cannot be used (see [MathJSON Symbols](/math-json/#symbols))                                      |
| `invalid-domain`               | the domain is not a valid domain literal or domain expression                                                    |
| `expected-closing-delimiter`   | a closing `}` was expected, but is missing                                                                       |
| `unexpected-closing-delimiter` | a closing `}` was encountered, but not expected                                                                  |
| `expected-environment-name`    | the name of an environment should be provided with a `\begin` or `\end` command                                  |
| `unknown-environment`          | the environment name provided cannot be parsed                                                                   |
| `unbalanced-environment`       | the named used with the `\begin` and `\end` commands should match                                                |
| `unexpected-operator`          | the operator does not apply to the current parsing context. Could be an infix or postfix operator without a rhs. |
| `unexpected-digit`             | the string included some characters outside of the range of expected digits                                      |
| `expected-string-argument`     | the argument was expected to be a string                                                                         |
| `unexpected-base`              | the base is outside of the expected range (2..36)                                                                |
| `iteration-limit-exceeded`     | a loop has reached the maximum iteration limit                                                                   |

</div>

```ts
console.log(ce.parse("\\oops").json);
// ➔ ["Error", ["ErrorCode","'unexpected-command'","'\\oops'"], ["Latex","'\\oops'"]

console.log(ce.parse("\\oops{bar}+2").json);
// ➔  ["Add",
//        ["Error",
//          ["ErrorCode","'unexpected-command'","'\\oops'"],
//          ["Latex","'\\oops{bar}'"]
//        ],
//        2
//    ]

console.log(ce.parse("\\begin{oops}\\end{oops}").json);
// ➔ ["Error",["ErrorCode","'unknown-environment'",""oops""],["Latex","'\\\\begin{oops}\\\\end{oops}'"]

console.log(ce.parse("1+\\sqrt").json);
// ➔ ["Add", 1 ,["Sqrt", ["Error", ""missing""]]]

console.log(ce.parse("1+\\frac{2}").json);
// ➔ ["Add", 1, ["Divide", 2, ["Error",""missing""]]]

console.log(ce.parse("1+(2=2)+2").json);
// ➔ ["Add", 1, ["Delimiter", ["Equal", 2, 2]]]

console.log(ce.parse("1+(2=2)+3").canonical.json);
// ➔ ["Add",
//      1,
//      ["Error",
//          ["ErrorCode", "'incompatible-domain'", "Numbers", "Booleans"],
//          ["Delimiter", ["Equal", 2, 2]]
//      ],
//      3
//    ]

console.log(ce.parse("\\times 3").json);
// ➔ ["Sequence", ["Error", ["ErrorCode", "'unexpected-operator'", "'\\times'"], ["Latex","'\\times'"]], 3]

console.log(ce.parse("x__+1").json);
// ➔ ["Add", ["Subscript", "x", ["Error","'syntax-error'", ["Latex","'_'"]]], 1]

console.log(ce.parse("x_{a").json);
// ➔ ["Subscript", "x", ["Error", "'expected-closing-delimiter'", ["Latex","'{a'"]]]


console.log(ce.parse("x@2").json);
// ➔ ["Sequence", "x", ["Error", ["ErrorCode", "'unexpected-token'", "'@'"], ["Latex", "'@2'"]]]
```
