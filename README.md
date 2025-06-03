# Proposal: try/catch expression with `raise`

## Author

Dmitriy Tarasov

Telegram: [@tarasov_d_a](https://t.me/tarasov_d_a)

GitHub: [dmitrytarassov](https://github.com/dmitrytarassov)

## License

© 2025 Dmitriy Tarasov
Licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) — free to use with attribution.

## Motivation

JavaScript lacks a convenient way to assign a value declaratively based on the outcome of a `try/catch` block, especially when used inside expressions like `if`, `switch`, or variable initialization.

Instead of verbose patterns like:

```js
let value;
try {
  value = doSomething();
} catch (e) {
  value = fallbackValue;
}
```

We propose a new syntax:

```js
const value = try {
  raise doSomething();
} catch (e) {
  raise fallbackValue;
}
```

## Proposed Syntax and Semantics

```js
const result = try {
  raise someFn();
} catch (e) {
  raise 'fallback';
} finally (raised) {
  if (raised === 'fallback') {
      console.log('used fallback')
  }
  raise raised;
}
```

### Core Principles:

* `raise` works like a `return` inside `try` or `catch`, but returns a value *from the expression*.
* If `raise` is used in either `try` or `catch`, it **must** appear in both.
* If no `raise` is used, the `try/catch/finally` block returns nothing.
* `finally (raised)` receives the raised value and **may override it** by calling `raise` again.
* `raise` can accept any value, including `undefined`.
* Works with `async/await`: `raise await ...` returns a resolved value.

## Usage Examples

### Declarative assignment:

```js
const user = try {
  raise getUserFromCache();
} catch {
  raise await fetchUser();
}
```

### Inside an `if`:

```js
if (try {
    raise validate(data);
  } catch {
    raise false
  }) {
  submit(data);
}
```

### In a `switch`:

```js
switch (try {
    raise compute();
  } catch {
    raise 'default'
  }) {
  case 'A': ...
  case 'default': ...
}
```

### With `finally`:

```js
const v = try {
  raise 1;
} catch {
  raise 2;
} finally (raised) {
  if (raised > 1) alert('big value');
  raise 3;
}
// v === 3
```

## Analogs in Other Languages

* **Python** — `try` is a statement, not an expression.
* **Kotlin** — `runCatching { ... }.getOrElse { ... }`
* **Scala** — `Try { ... }.getOrElse(...)`
* **Rust** — `Result<T, E>` with `?` operator

## Implementation Considerations

* `raise` becomes a new reserved keyword — potential conflicts.
* `raised` in `finally` has explicit block scope.
* Works with `async/await`, `yield`, and `return` in generators.
* Interaction with TDZ, hoisting, linters.
* AST rewriting and parser upgrades required.

## Alternatives Considered

* Implicit return of the last expression — too ambiguous.
* `try { return ... } catch { return ... }` — invalid in expression context.

## Transpilation Possibility

Example transformation via Babel:

```js
const result = try {
  raise doA();
} catch (e) {
  raise doB();
}
```

becomes:

```js
let __raised;
try {
  __raised = doA();
} catch (e) {
  __raised = doB();
}
const result = __raised;
```

## Conclusion

`raise` expressions offer a powerful way to make JavaScript more expressive and functional in style without sacrificing readability or compatibility. This proposal introduces an elegant, composable control-flow construct that cleanly extends the semantics of `try/catch`.

---

*This is a draft proposal. Feedback, discussion, and contributions are welcome.*
