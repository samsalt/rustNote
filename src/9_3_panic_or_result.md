# Notes on Error Handling in Rust: `panic!` vs. `Result`

## Core Philosophy

In Rust, error handling is divided into two main categories:

1.  **Recoverable Errors**: Errors that can be reasonably handled, like a file not being found. These are handled with the `Result<T, E>` enum.
2.  **Unrecoverable Errors**: Errors that represent bugs or states from which the program cannot recover, like accessing an array index out of bounds. These are handled with the `panic!` macro, which stops the program.

By default, it's better to return a `Result` because it gives the code that calls your function the power to decide how to handle the error. They can attempt to recover or choose to `panic!` themselves.

## When to `panic!`

While `Result` is the default, there are specific scenarios where `panic!` is more appropriate.

### 1\. Examples, Prototyping, and Tests

  - **Examples**: Using extensive error handling can make example code harder to understand. Methods like `.unwrap()` and `.expect()` (which call `panic!` on an `Err` value) are often used as placeholders.
  - **Prototyping**: When you're still designing your application, `unwrap()` and `expect()` leave clear markers in your code for where you need to add more robust error handling later.
  - **Tests**: If an operation in a test fails, you want the entire test to fail. `panic!` is the mechanism that marks a test as failed, so using `unwrap()` or `expect()` is the correct approach.

### 2\. Cases Where You Know More Than the Compiler

Sometimes, you can logically guarantee that a `Result` will always be `Ok`, even if the compiler can't. In these situations, using `expect()` is acceptable, especially if you document the assumption.

**Example:**

```rust
use std::net::IpAddr;

// We know this is a valid IP, so a panic is not expected.
let home: IpAddr = "127.0.0.1"
    .parse()
    .expect("Hardcoded IP address should be valid");
```

### 3\. Guidelines for Library Code

Your code should `panic!` when it could end up in a "bad state." A bad state is when some core assumption, guarantee, or invariant has been broken. This is usually true if one or more of the following apply:

  - The bad state is completely unexpected.
  - Your code after this point must rely on not being in this bad state.
  - There isn’t a good way to encode this state information in the types you're using.
  - **Contract Violation**: If a function's contract is violated (e.g., it receives invalid arguments), it indicates a bug on the caller's side. Panicking makes sense because there's no reasonable way for the calling code to recover; the programmer needs to fix their code.

## When to Return `Result`

Return a `Result` when failure is an expected possibility that the calling code should be prepared to handle.

  - A parser being given malformed data.
  - An HTTP request failing due to a rate limit.
  - An operation that might fail for reasons outside the program's control.

## Using the Type System for Validation

Instead of adding checks everywhere, you can use Rust's type system to enforce validity.

### Creating Custom Types for Validation

A powerful technique is to create a new type that can only be instantiated with valid data. Functions can then accept this type in their signature, guaranteeing they receive valid input without needing to perform checks themselves.

**Example:** A `Guess` type that ensures a value is between 1 and 100.

```rust
pub struct Guess {
    // The value is private to prevent direct modification
    value: i32,
}

impl Guess {
    // The only way to create a Guess is through `new()`
    pub fn new(value: i32) -> Guess {
        if value < 1 || value > 100 {
            // If validation fails, panic! This is a contract violation.
            panic!("Guess value must be between 1 and 100, got {value}.");
        }
        Guess { value }
    }

    // A "getter" method to access the value
    pub fn value(&self) -> i32 {
        self.value
    }
}
```

Now, any function that takes a `Guess` as a parameter can proceed with confidence, knowing the value is guaranteed to be between 1 and 100.