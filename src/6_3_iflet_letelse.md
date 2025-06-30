### Concise Control Flow in Rust: `if let` and `let else`

This note summarizes the use of `if let` and `let...else` for more concise control flow in Rust, as an alternative to the more verbose `match` statement.

#### The `if let` Expression

The `if let` syntax is a less verbose way to handle a value that matches a single pattern while ignoring all other possible patterns.

**When to use it:**

  * When you are only interested in one variant of an `enum` or `Option` and want to ignore the rest.
  * To avoid the boilerplate of a `match` statement where most arms would be empty or do nothing.

**Example:**

Instead of this `match` statement:

```rust
let config_max = Some(3u8);
match config_max {
    Some(max) => println!("The maximum is configured to be {max}"),
    _ => (),
}
```

You can use the more concise `if let`:

```rust
let config_max = Some(3u8);
if let Some(max) = config_max {
    println!("The maximum is configured to be {max}");
}
```

You can also include an `else` block with `if let`, which is equivalent to the `_` arm in a `match` expression.

**Example with `else`:**

```rust
let mut count = 0;
if let Coin::Quarter(state) = coin {
    println!("State quarter from {state:?}!");
} else {
    count += 1;
}
```

-----

### The `let...else` Expression

The `let...else` syntax is useful for destructuring a value and continuing execution if it matches, or diverging (e.g., returning from the function) if it doesn't. This helps in keeping the "happy path" of your code less nested.

**When to use it:**

  * When you want to extract a value from a pattern and use it in the rest of the function.
  * When a non-match should immediately exit the current scope (e.g., by returning, breaking, or continuing).

**Key characteristics:**

  * It takes a pattern on the left and an expression on the right.
  * If the pattern matches, the variables are bound and execution continues in the same scope.
  * If the pattern does *not* match, the `else` block is executed, which **must** diverge (e.g., `return`, `break`, `continue`).

**Example:**

This code uses `let...else` to ensure we have a `Coin::Quarter` before proceeding.

```rust
fn describe_state_quarter(coin: Coin) -> Option<String> {
    let Coin::Quarter(state) = coin else {
        return None;
    };

    if state.existed_in(1900) {
        Some(format!("{state:?} is pretty old, for America!"))
    } else {
        Some(format!("{state:?} is relatively new."))
    }
}
```