# Places Where Patterns Are Used in Rust

Patterns are a special syntax in Rust for matching against the structure of types. They can be used in various places throughout the language.

### 1\. `match` Arms

Patterns are a fundamental part of `match` expressions, used to control the flow of your program based on a value.

  * **Syntax**:
    ```rust
    match VALUE {
        PATTERN => EXPRESSION,
        PATTERN => EXPRESSION,
        PATTERN => EXPRESSION,
    }
    ```
  * **Exhaustiveness**: `match` expressions must be exhaustive, meaning all possible values of the input type must be handled. The special `_` pattern can be used as a catch-all for any case not explicitly handled.
  * **Example**:
    ```rust
    match x {
        None => None,
        Some(i) => Some(i + 1),
    }
    ```

### 2\. Conditional `if let` Expressions

`if let` provides a concise way to handle a single pattern from a `match` and ignore the rest. It can be combined with `else` and `else if let`.

  * **Benefit**: More flexible than a `match` expression for complex, unrelated conditions.
  * **Drawback**: The compiler does not check for exhaustiveness as it does with `match`.
  * **Example**:
    ```rust
    let favorite_color: Option<&str> = None;
    let is_tuesday = false;
    let age: Result<u8, _> = "34".parse();

    if let Some(color) = favorite_color {
        println!("Using your favorite color, {color}, as the background");
    } else if is_tuesday {
        println!("Tuesday is green day!");
    } else if let Ok(age) = age {
        if age > 30 {
            println!("Using purple as the background color");
        } else {
            println!("Using orange as the background color");
        }
    } else {
        println!("Using blue as the background color");
    }
    ```

### 3\. `while let` Conditional Loops

A `while let` loop runs as long as a pattern continues to match a value.

  * **Use Case**: Commonly used for processing items from an iterator or a channel until a specific "end" value is received.
  * **Example**: Receiving messages from a channel until it's empty and disconnected.
    ```rust
    // Assuming `rx` is the receiver end of a channel
    while let Ok(value) = rx.recv() {
        println!("{value}");
    }
    ```

### 4\. `for` Loops

In a `for` loop, the item that directly follows the `for` keyword is a pattern. This is very useful for destructuring compound values from an iterator.

  * **Example**: Destructuring a tuple from the `.enumerate()` method.
    ```rust
    let v = vec!['a', 'b', 'c'];

    for (index, value) in v.iter().enumerate() {
        println!("{value} is at index {index}");
    }
    ```

### 5\. `let` Statements

Every `let` statement is actually a form of pattern matching.

  * **Syntax**: `let PATTERN = EXPRESSION;`
  * **Simple Case**: In `let x = 5;`, `x` is a pattern that binds to the value `5`.
  * **Destructuring**: `let` can be used to break apart a tuple, struct, or enum.
    ```rust
    let (x, y, z) = (1, 2, 3);
    ```
  * **Refutability**: Patterns in `let` statements must be *irrefutable*, meaning they must match for any possible value of the expression. For example, `let Some(x) = an_option;` would fail to compile because `an_option` could be `None`.

### 6\. Function Parameters

Function and closure parameters can also be patterns, allowing for destructuring directly in the signature.

  * **Example**: Destructuring a tuple passed as an argument.
    ```rust
    fn print_coordinates(&(x, y): &(i32, i32)) {
        println!("Current location: ({x}, {y})");
    }

    fn main() {
        let point = (3, 5);
        print_coordinates(&point); // Prints "Current location: (3, 5)"
    }
    ```