Rust's declarative macros, often called "macros by example," are a powerful feature for metaprogramming, allowing you to write code that writes other code. This reduces boilerplate and creates domain-specific languages (DSLs) within Rust. Here's how to write them using the `macro_rules!` system.

### The Basics: `macro_rules!`

Declarative macros are defined with `macro_rules!`. The structure is similar to a `match` expression. It takes a series of rules, each with a "matcher" and a "transcriber," separated by `=>`.

A simple macro looks like this:

```rust
macro_rules! say_hello {
    () => {
        println!("Hello, World!");
    };
}

fn main() {
    say_hello!();
}
```

In this example, `()` is the matcher. It matches an empty argument list. The code within the `{}` is the transcriber, which is the code that will be generated when the macro is called.

-----

### Matchers and Transcribers

Macros shine when they can take arguments. You can define patterns in the matcher to capture different inputs. These captured values are then used in the transcriber to generate code.

#### Fragment Specifiers

To match different kinds of Rust code, you use "fragment specifiers." These are like types for macro arguments. Some common ones include:

  * `ident`: an identifier (like a variable or function name).
  * `expr`: an expression.
  * `ty`: a type.
  * `stmt`: a statement.
  * `pat`: a pattern.
  * `block`: a block of code (enclosed in `{}`).
  * `item`: an item (like a struct, enum, or function).
  * `tt`: a token tree (can represent almost anything).

Here's a macro that takes an expression and prints it:

```rust
macro_rules! print_expr {
    ($e:expr) => {
        println!("{:?} = {:?}", stringify!($e), $e);
    };
}

fn main() {
    print_expr!(1 + 2);
}
```

In this macro:

  * `$e:expr` matches any expression and assigns it to the variable `$e`.
  * In the transcriber, `$e` is used twice: once with `stringify!` to get the literal expression as a string, and once to evaluate the expression.

-----

### Repetition

A key feature of declarative macros is the ability to handle a variable number of arguments using repetition syntax. This is done with `$()*`, `$()+`, or `$()?`.

  * `$()*`: Matches zero or more repetitions.
  * `$()+`: Matches one or more repetitions.
  * `$()?`: Matches an optional element (zero or one).

You can also specify a separator between repetitions.

Here is a simplified version of the `vec!` macro:

```rust
macro_rules! my_vec {
    ( $( $x:expr ),* ) => {
        {
            let mut temp_vec = Vec::new();
            $(
                temp_vec.push($x);
            )*
            temp_vec
        }
    };
}

fn main() {
    let v = my_vec![1, 2, 3];
    println!("{:?}", v);
}
```

In this example:

  * `$( $x:expr ),*` matches zero or more expressions, separated by commas.
  * The transcriber uses `$( temp_vec.push($x); )*` to generate a `push` statement for each matched expression.

-----

### Multiple Rules

Macros can have multiple rules, and they are matched in the order they are defined. This is useful for creating macros that can be called in different ways.

Let's expand our `my_vec!` macro to handle creating a vector with a repeated value, similar to the standard library's `vec![value; count]`.

```rust
macro_rules! my_vec {
    // Rule for a list of expressions
    ( $( $x:expr ),* ) => {
        {
            let mut temp_vec = Vec::new();
            $(
                temp_vec.push($x);
            )*
            temp_vec
        }
    };
    // Rule for a single value repeated
    ( $elem:expr; $n:expr ) => {
        std::vec::from_elem($elem, $n)
    };
}

fn main() {
    let v1 = my_vec![1, 2, 3];
    let v2 = my_vec![0; 5];
    println!("v1: {:?}", v1);
    println!("v2: {:?}", v2);
}
```

The macro will first try to match the `( $( $x:expr ),* )` rule. If that fails, it will then try to match the `( $elem:expr; $n:expr )` rule.

-----

### Macro Hygiene

Rust macros are largely "hygienic." This means that variables introduced within a macro will not conflict with variables in the scope where the macro is called.

However, when your macro needs to refer to an item (like a function or a module) from the crate in which the macro is defined, you should use the special `$crate` variable. This ensures that even if the macro is used in another crate, it can still find the items from its home crate.

```rust
// In your library crate (e.g., my_library)
#[macro_export]
macro_rules! use_helper {
    () => {
        $crate::helper_function();
    };
}

pub fn helper_function() {
    println!("Called helper function!");
}
```

-----

### Exporting and Using Macros

To make a macro available to other modules or crates, you need to use the `#[macro_export]` attribute.

```rust
#[macro_export]
macro_rules! my_macro {
    // ... rules ...
}
```

When you import a crate that exports macros, you can use them directly. For using macros from other modules within the same crate, you can bring them into scope with a `use` statement.

```rust
// In another crate that depends on my_library
use my_library::use_helper;

fn main() {
    use_helper!();
}
```

By understanding these core concepts, you can start writing powerful and expressive declarative macros in Rust to simplify your code and enhance its functionality.