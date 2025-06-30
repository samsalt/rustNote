# Improving an I/O Project with Iterators in Rust

This note summarizes how to refactor a Rust I/O project by leveraging the power and conciseness of iterators, as detailed in Chapter 13 of the Rust Programming Language book.

## 1\. Refactoring `Config::build` to Use an Iterator

The goal is to make the argument parsing more efficient by taking ownership of an iterator, thus avoiding unnecessary memory allocations from `.clone()`.

### Before: Using a Slice and `clone()`

The original `Config::build` function took a slice of `String`s and had to clone the arguments to take ownership of them.

```rust
// Filename: src/lib.rs

impl Config {
    pub fn build(args: &[String]) -> Result<Config, &'static str> {
        if args.len() < 3 {
            return Err("not enough arguments");
        }
        let query = args[1].clone();
        let file_path = args[2].clone();
        let ignore_case = env::var("IGNORE_CASE").is_ok();

        Ok(Config {
            query,
            file_path,
            ignore_case,
        })
    }
}
```

### After: Taking Ownership of an Iterator

The code is refactored to pass the iterator from `env::args()` directly to `Config::build`.

**Changes in `main.rs`:**

The `.collect()` call is removed. The iterator returned by `env::args()` is passed directly.

```rust
// Filename: src/main.rs

fn main() {
    let config = Config::build(env::args()).unwrap_or_else(|err| {
        eprintln!("Problem parsing arguments: {err}");
        process::exit(1);
    });

    // --snip--
}
```

**Changes in `lib.rs`:**

The `Config::build` function signature is updated to accept any type that implements `Iterator` with `String` items. The function body then uses the `.next()` method to consume the iterator and extract the arguments.

```rust
// Filename: src/lib.rs

impl Config {
    pub fn build(
        mut args: impl Iterator<Item = String>,
    ) -> Result<Config, &'static str> {
        args.next(); // Skip the program name

        let query = match args.next() {
            Some(arg) => arg,
            None => return Err("Didn't get a query string"),
        };

        let file_path = match args.next() {
            Some(arg) => arg,
            None => return Err("Didn't get a file path"),
        };

        let ignore_case = env::var("IGNORE_CASE").is_ok();

        Ok(Config {
            query,
            file_path,
            ignore_case,
        })
    }
}
```

## 2\. Refactoring `search` with Iterator Adapters

The `search` function can be made more concise and functional by using iterator adapters like `.filter()` and `.collect()`.

### Before: Using a `for` loop

The original implementation used a `for` loop and a mutable vector to store results.

```rust
// Filename: src/lib.rs

pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    let mut results = Vec::new();

    for line in contents.lines() {
        if line.contains(query) {
            results.push(line);
        }
    }

    results
}
```

### After: Using Iterator Adapters

This version chains iterator methods for a more declarative style. It creates an iterator from the lines, filters them based on the query, and collects the results into a new vector.

```rust
// Filename: src/lib.rs

pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    contents
        .lines()
        .filter(|line| line.contains(query))
        .collect()
}
```

## Choosing Between Loops and Iterators

  * **Iterator Style**: Most Rust programmers prefer this style. It's often more readable and concise once you are familiar with the common iterator adapters. It abstracts away the manual loop management and focuses on the high-level objective.
  * **Loop Style**: Can feel more straightforward for beginners.

While the intuitive assumption is that a lower-level loop might be faster, the performance of iterators in Rust is a key consideration.