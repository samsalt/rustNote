# Understanding Unrecoverable Errors with `panic!` in Rust

In Rust, the `panic!` macro is used for errors from which there is no clean recovery. It signifies that the program is in a state where it cannot continue.

## Triggering a Panic

A panic can be triggered in two main ways:

1.  **Implicitly**: By performing an action that causes the code to fail, such as accessing an array index that is out of bounds.
2.  **Explicitly**: By directly calling the `panic!` macro.

<!-- end list -->

```rust
fn main() {
    // Explicit panic
    panic!("crash and burn");
}
```

## Responding to a Panic: Unwinding vs. Aborting

When a `panic!` occurs, Rust's default behavior is to **unwind** the stack.

  * **Unwinding**: Rust walks back up the stack, cleaning up the data from each function it encounters. This is the default because it ensures a safe but slower shutdown.
  * **Aborting**: You can choose to immediately abort the program without cleanup. The operating system then reclaims the memory. This results in a smaller binary.

To change the panic behavior to `abort` for a release build, modify your `Cargo.toml` file:

```toml
[profile.release]
panic = 'abort'
```

## Reading Panic Output

When a panic occurs, the console displays an error message pointing to the file, line, and character where the issue occurred.

### Example: Index Out of Bounds

If you try to access a non-existent index in a vector, Rust will panic to prevent a buffer overread, which is a significant security vulnerability in other languages like C.

```rust
fn main() {
    let v = vec![1, 2, 3];
    v[99]; // This will panic
}
```

The output will be:

> thread 'main' panicked at src/main.rs:4:5:
> index out of bounds: the len is 3 but the index is 99

## Using a Backtrace to Find the Cause

For more complex panics, especially those originating in external libraries, a backtrace is essential for debugging. A backtrace is a list of all the functions that were called leading up to the panic.

To enable backtraces, run your program with the `RUST_BACKTRACE` environment variable set to `1`.

```shell
$ RUST_BACKTRACE=1 cargo run
```

When reading the backtrace, start from the top and look for the first entry that points to a file you wrote. This is typically the origin of the problem. Note that debug symbols must be enabled to get a useful backtrace, which is the default for `cargo build` and `cargo run` (but not for `cargo run --release`).