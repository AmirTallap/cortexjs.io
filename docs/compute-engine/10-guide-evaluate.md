---
title: Evaluation of Expressions
slug: /compute-engine/guides/evaluate/
preamble:
---

# Evaluation

<Intro>
Evaluating an expression is the process of determining the value of the
expression. This involves looking up the definitions of symbols and functions,
evaluating the arguments of functions, and applying the function to the
arguments.
</Intro>

## Evaluation Methods

**To evaluate an expression**, use the `expr.evaluate()` method.

```live
const expr = ce.parse('2 + 2');
expr.evaluate().print();
```

### Numeric Approximation

By default, `expr.evaluate()` preserves [exact values](/compute-engine/guides/numeric-evaluation/#exact-evaluation) in the result.
To force [numeric evaluation](/compute-engine/guides/numeric-evaluation/) use the `numericApproximation` option.

```live
const expr = ce.parse('2\\pi');
expr.evaluate().print();
expr.evaluate({ numericApproximation: true }).print();
```

The `expr.N()` method is a shorthand for `expr.evaluate({numericApproximation: true})`.

```live
const expr = ce.parse('2\\pi');
expr.N().print();
```

### Compilation

An expression can be evaluated by compiling it to JavaScript using the `expr.compile()` method.

<ReadMore path="/compute-engine/guides/compiling/" > 
Read more about **compiling expressions** <Icon name="chevron-right-bold" />
</ReadMore>

## Asynchronous Evaluation

Some computations can be time-consuming. For example, computing a very large
factorial. To prevent the browser from freezing, the Compute Engine can
perform some operations asynchronously.

**To perform an asynchronous evaluation**, use the `expr.evaluateAsync()` method.

```live
try {
  const fact = ce.parse('(70!)!');
  const factResult = await fact.evaluateAsync();
  factResult.print();
} catch (e) {
  console.error(e);
}
```

The `expr.evaluateAsync()` method returns a `Promise` that resolves to the result
of the evaluation. It accepts the same `numericApproximation` options as `expr.evaluate()`.

It is also possible to interrupt an evaluation, for example by providing the user
with a pause/cancel button. 

**To make an evaluation interruptible**, use an `AbortController`
object and a `signal`. 

For example, to interrupt an evaluation after 500ms:

```live
const abort = new AbortController();
const signal = abort.signal;
setTimeout(() => abort.abort(), 500);
try {
  const fact = ce.parse('(70!)!');
  const factResult = await fact.evaluateAsync({ signal });
  factResult.print();
} catch (e) {
  console.error(e);
}
```

```live
:::html
<div class="stack">
  <div class="row">
    <button id="evaluate-button">Evaluate</button>
    <button id="cancel-button" disabled>Cancel</button>
  </div>

  <div id="output"></div>
</div>
:::js
const abort = new AbortController();

document.getElementById('cancel-button').addEventListener('click', 
  () => abort.abort()
);

document.getElementById('evaluate-button').addEventListener('click', async () => {
  try {
    document.getElementById('evaluate-button').disabled = true;
    document.getElementById('cancel-button').disabled = false;

    const fact = ce.parse('(70!)!');
    const factResult = await fact.evaluateAsync({ signal: abort.signal });
    document.getElementById('output').textContent = factResult.toString();
    
    document.getElementById('evaluate-button').disabled = false;
    document.getElementById('cancel-button').disabled = true;
  } catch (e) {
    document.getElementById('evaluate-button').disabled = false;
    document.getElementById('cancel-button').disabled = true;
    console.error(e);
  }
});

```



**To set a time limit for an operation**, use the `ce.timeLimit` option, which
is a number of milliseconds.

```live
ce.timeLimit = 1000;
try {
  const fact = ce.parse('(70!)!');
  fact.evaluate().print();
} catch (e) {
  console.error(e);
}
```

The time limit applies to both the synchronous or asynchronous evaluation.

The default time limit is 2,000ms (2 seconds).

When an operation is canceled either because of a timeout or an abort, a
`CancellationError` is thrown.



## Lexical Scopes and Evaluation Contexts

The Compute Engine supports
[lexical scoping](<https://en.wikipedia.org/wiki/Scope_(computer_science)>).

A **lexical scope** is a table mapping identifiers to their definitions. The 
identifiers are the names of the symbols and functions in the scope.

The lexical scope resolves metadata about identifiers, such as their type and 
constant status.

An **evaluation context** is a snapshot of the current state of the compute 
engine. It includes the values of all identifiers and the current scope.

Evaluation contexts are arranged in a stack, with the current (top-most) 
evaluation context available with `ce.context`.

Evaluation contexts are created automatically, for example when a new scope is
created, or each time a recursive function is called.

### Binding

**[Name Binding](https://en.wikipedia.org/wiki/Name_binding) is the process of
associating an identifier (the name of a function or symbol) with a
definition.**

Name Binding should not be confused with **value binding** which is the process
of associating a **value** to a symbol.

<ReadMore path="/compute-engine/guides/symbols/#scopes" >Read more about
<strong>identifiers</strong> and value binding.<Icon name="chevron-right-bold" /></ReadMore>

To bind a name to a definition, the Compute Engine looks up the symbol table
of the current scope. If the identifier is not found, the parent scope is
searched, and so on until a definition is found.

If no definition is found, the identifier is declared as a symbol of type
`unknown`, or a more specific type if the context allows it.

If the identifier is found, the definition record is used to evaluate the
expression. 

The definition record contains information about the identifier, such as its
type, whether it is a constant, and how to evaluate it.

There are two kind of definition records:
1. **Value Definition Record**: This record contains information about the
   symbol, such as its type and whether it is a constant.
2. **Operator Definition Record**: This record contains information about the
   operator, such as its signature and how to evaluate it.

Name binding is done during canonicalization. If name binding failed, the
`isValid` property of the expression is `false`.

**To get a list of the errors in an expression** use the `expr.errors` property.

<ReadMore path="/compute-engine/guides/expressions/#errors" > Read more about the
<strong>errors</strong> <Icon name="chevron-right-bold" /></ReadMore>


### Creating New Scopes

**To add a new scope** use `ce.pushScope()`.

```ts
ce.assign('x', 100); // "x" is defined in the current scope
ce.pushScope();
ce.assign('x', 500); // "x" is defined in the new scope
ce.box('x').print(); // 500
ce.popScope();
ce.box('x').print(); // 100
```

**To exit a scope** use `ce.popScope()`.

This will invalidate any definitions associated with the scope, and restore the
symbol table from previous scopes that may have been shadowed by the current
scope.


## Evaluation Loop

:::info

This is an advanced topic. You don't need to know the details of how the
evaluation loop works, unless you're interested in extending the standard
library and providing your own function definitions.

:::

Each identifier (name of symbol or function) is **bound** to a definition within
a **lexical scope** during canonicalization. This usually happens when calling
`ce.box()` or `ce.parse()`, or if accessing the `.canonical` property of a
non-canonical expression.

When a function is evaluated, the following steps are followed:

1. If the expression is not canonical, it cannot be evaluated and an error is
   thrown. The expression must be canonicalized first.

2. Each argument of the function are evaluated, left to right, unless the 
   function has a `lazy` flag. If the function is lazy, the arguments are not
   evaluated.

   1. An argument can be **held**, in which case it is not evaluated. Held
      arguments can be useful when you need to pass a symbolic expression to a
      function. If it wasn't held, the result of evaluating the expression would
      be used, not the symbolic expression.

      Alternatively, using the `Hold` function will prevent its argument from
      being evaluated. Conversely, the `ReleaseHold` function will force an
      evaluation.

   2. If an argument is a `["Sequence"]` expression, treat each argument of the
      sequence expression as if it was an argument of the function. If the
      sequence is empty, ignore the argument.

3. If the function is associative, flatten its arguments as necessary. \\[
   f(f(a, b), c) \to f(a, b, c) \\]

4. Apply the function to the arguments

5. Return the canonical form of the result
