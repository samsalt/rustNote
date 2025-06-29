# Notes on Recoverable Errors with `Result` in Rust

This note summarizes how to handle recoverable errors in Rust using the `Result<T, E>` enum.

## 1\. The `Result` Enum

Rust handles recoverable errors with the `Result<T, E>` enum. It has two variants:

  * `Ok(T)`: Represents success and contains a value of type `T`.
  * `Err(E)`: Represents an error and contains an error value of type `E`.

<!-- end list -->

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

**Example:** The `File::open("hello.txt")` function returns a `Result<std::fs::File, std::io::Error>`.

## 2\. Handling `Result`

### Using a `match` Expression

The most basic way to handle a `Result` is with a `match` expression.

```rust
use std::fs::File;

fn main() {
    let greeting_file_result = File::open("hello.txt");

    let greeting_file = match greeting_file_result {
        Ok(file) => file,
        Err(error) => panic!("Problem opening the file: {:?}", error),
    };
}
```

### Matching on Different Kinds of Errors

You can use a nested `match` to handle specific errors, like a file not being found.

```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let greeting_file_result = File::open("hello.txt");

    let greeting_file = match greeting_file_result {
        Ok(file) => file,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Problem creating the file: {:?}", e),
            },
            _ => {
                panic!("Problem opening the file: {:?}", error);
            }
        },
    };
}
```

## 3\. Shortcuts for Handling Errors

### Methods for Panic on Error: `unwrap` and `expect`

  * **`unwrap()`**: A shortcut method that returns the value inside `Ok`. If the `Result` is `Err`, it calls `panic!`.

    ```rust
    use std::fs::File;
    let greeting_file = File::open("hello.txt").unwrap();
    ```

  * **`expect(message)`**: Similar to `unwrap`, but lets you provide a custom panic message. This is generally preferred in production code for better error context.

    ```rust
    use std::fs::File;
    let greeting_file = File::open("hello.txt")
        .expect("hello.txt should be included in this project");
    ```

### Closures: `unwrap_or_else`

For more complex logic without nesting `match`, you can use combinators and closures. `unwrap_or_else` executes a closure on an `Err` value.

```rust
use std::fs::File;
use std::io::ErrorKind;

let greeting_file = File::open("hello.txt").unwrap_or_else(|error| {
    if error.kind() == ErrorKind::NotFound {
        File::create("hello.txt").unwrap_or_else(|error| {
            panic!("Problem creating the file: {:?}", error);
        })
    } else {
        panic!("Problem opening the file: {:?}", error);
    }
});
```

## 4\. Propagating Errors

Instead of handling an error within a function, you can return it to the calling code. This is known as *propagating* the error.

### The Question Mark `?` Operator

The `?` operator is a shortcut for propagating errors. It makes code much cleaner. If a `Result` is `Ok`, the expression returns the value inside the `Ok`. If it's `Err`, the `Err` is returned from the entire function.

**Manual Propagation with `match`:**

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let username_file_result = File::open("hello.txt");

    let mut username_file = match username_file_result {
        Ok(file) => file,
        Err(e) => return Err(e),
    };

    let mut username = String::new();

    match username_file.read_to_string(&mut username) {
        Ok(_) => Ok(username),
        Err(e) => Err(e),
    }
}
```

**Simplified Code with the `?` Operator:**

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let mut username_file = File::open("hello.txt")?;
    let mut username = String::new();
    username_file.read_to_string(&mut username)?;
    Ok(username)
}
```

This can be chained for even more concise code:

```rust
use std::fs;
use std::io;

fn read_username_from_file() -> Result<String, io::Error> {
    fs::read_to_string("hello.txt")
}
```

### Where to Use the `?` Operator

The `?` operator can only be used in functions that return a `Result`, `Option`, or another type that implements `FromResidual`. You cannot use it in a `main` function that returns `()`.

### Using `Result` in `main`

The `main` function can return a `Result`. This allows you to use the `?` operator within it.

```rust
use std::error::Error;
use std::fs::File;

fn main() -> Result<(), Box<dyn Error>> {
    let greeting_file = File::open("hello.txt")?;
    Ok(())
}
```

  * `Box<dyn Error>` is a trait object that means "any kind of error".
  * If `main` returns `Ok(())`, the program exits with code `0`.
  * If `main` returns an `Err` value, it exits with a non-zero error code.