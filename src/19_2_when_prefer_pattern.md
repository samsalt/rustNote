# Refutability in Rust Patterns

In Rust, patterns are categorized into two types: refutable and irrefutable.

#### Irrefutable Patterns
- An **irrefutable pattern** is a pattern that will always match any possible value.
- A prime example is the variable `x` in a `let` statement like `let x = 5;`. Here, `x` will match anything, so it cannot fail.
- You can only use irrefutable patterns in:
    - Function parameters
    - `let` statements
    - `for` loops

#### Refutable Patterns
- A **refutable pattern** is a pattern that can fail to match for some possible values.
- For example, in `if let Some(x) = a_value`, the pattern `Some(x)` is refutable because it will not match if `a_value` is `None`.
- You can use refutable patterns in:
    - `if let` expressions
    - `while let` expressions
    - `let...else` statements

#### Handling Mismatches
The compiler will issue a warning if you use an irrefutable pattern where a refutable one is expected because the conditional aspect of the expression becomes unnecessary.

If you attempt to use a refutable pattern in a context that requires an irrefutable one, such as `let Some(x) = some_option_value;`, Rust will produce a compiler error. This is because the code has no valid way to proceed if the pattern doesn't match.

To resolve this, you can change the construct you are using. For instance, you could switch from `let` to `if let` or `let...else` to provide a path for the code to follow in case of a non-match.