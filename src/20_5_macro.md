Of course\! Here is a structured markdown note based on the provided web page about Rust macros.

-----

## 🦀 Rust Macros: A Comprehensive Overview

Macros in Rust are a powerful feature for **metaprogramming**, which is the practice of writing code that writes other code. They help reduce boilerplate and add functionality that regular functions cannot provide.

### The Difference Between Macros and Functions

| Feature | Macros | Functions |
| :--- | :--- | :--- |
| **Arguments** | Can take a variable number of parameters. | Must declare a fixed number and type of parameters. |
| **Execution Time**| Expanded at **compile time**. | Called at **runtime**. |
| **Capabilities** | Can implement traits on types, define new syntax. | Cannot implement traits or modify language syntax. |
| **Complexity** | More complex to define and maintain. | Generally simpler to read, write, and understand. |
| **Definition** | Must be defined or brought into scope *before* they are called. | Can be defined anywhere in any order. |

-----

## Types of Macros

There are two main families of macros in Rust: **Declarative** and **Procedural**.

### 1\. Declarative Macros (`macro_rules!`)

Also known as "macros by example," these are the most common type of macro. They work by matching against patterns in the input code and replacing them with new code.

  * **Definition**: Uses the `macro_rules!` construct.

  * **Syntax**: Similar to a `match` expression, where patterns are compared against the structure of the provided Rust code.

  * **Example**: A simplified version of the `vec!` macro.

    ```rust
    #[macro_export]
    macro_rules! vec {
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
    ```

### 2\. Procedural Macros

Procedural macros act more like functions that operate on Rust code. They take a stream of tokens as input, process it, and produce a new stream of tokens as output. They must be defined in their own special crate type.

There are three kinds of procedural macros:

#### a. Custom `#[derive]` Macros

These macros allow you to create your own `derive` attributes to automatically implement traits for structs and enums.

  * **Purpose**: Automatically generate trait implementations.
  * **Annotation**: `#[proc_macro_derive(YourTraitName)]`
  * **Key Crates**:
      * `proc_macro`: The compiler's API for manipulating Rust code.
      * `syn`: For parsing Rust code from a token stream into a syntax tree.
      * `quote`: For turning a syntax tree back into a token stream.

#### b. Attribute-like Macros

These are more flexible than `derive` macros. They allow you to create new, custom attributes that can be applied to almost any item, including functions.

  * **Purpose**: Define custom attributes for any item.
  * **Example**: A `#[route(GET, "/")]` attribute for a web framework.
  * **Annotation**: `#[proc_macro_attribute]`

#### c. Function-like Macros

These macros look like function calls but operate on the tokens passed to them.

  * **Purpose**: Define flexible, function-call-like macros. They are more powerful than `macro_rules!` macros.
  * **Example**: An `sql!()` macro that can parse and validate an SQL query at compile time.
  * **Annotation**: `#[proc_macro]`