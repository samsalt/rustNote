### Understanding Rust Closures

Closures in Rust are anonymous functions that can be stored in variables, passed as arguments to other functions, and can capture values from the scope in which they are defined.

#### Key Characteristics:

  * **Anonymous:** They don't have a function name.
  * **Environment Capture:** They can capture variables from their surrounding scope. This can be done in three ways:
      * Immutably borrowing (`Fn`)
      * Mutably borrowing (`FnMut`)
      * Taking ownership (`FnOnce`)
  * **Type Inference:** The compiler can usually infer the types of the parameters and the return value, so explicit type annotations are often not necessary.

#### Syntax

The basic syntax for a closure uses vertical pipes `| |` for the parameters and braces `{ }` for the body.

```rust
// A closure with type annotations
let add_one = |x: u32| -> u32 {
    x + 1
};

// A more concise version with type inference
let add_one_inferred = |x| x + 1;
```

#### The `Fn` Traits

The way a closure captures and interacts with its environment is determined by the `Fn` traits:

  * **`FnOnce`**: Applies to any closure that can be called at least once. All closures implement this trait. Closures that move captured values out of their body only implement `FnOnce`.
  * **`FnMut`**: For closures that can mutate captured values but don't move them out of their body. These can be called more than once.
  * **`Fn`**: For closures that neither mutate nor move captured values. These can also be called multiple times.

#### Practical Application

A common use case for closures is with the `unwrap_or_else` method on `Option<T>`, where the closure provides a default value if the `Option` is `None`.

```rust
#[derive(Debug, PartialEq, Copy, Clone)]
enum ShirtColor {
    Red,
    Blue,
}

struct Inventory {
    shirts: Vec<ShirtColor>,
}

impl Inventory {
    fn giveaway(&self, user_preference: Option<ShirtColor>) -> ShirtColor {
        user_preference.unwrap_or_else(|| self.most_stocked())
    }

    fn most_stocked(&self) -> ShirtColor {
        // ... logic to find the most stocked shirt
        ShirtColor::Blue // Simplified for example
    }
}
```