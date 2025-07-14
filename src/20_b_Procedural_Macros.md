## Writing Procedural Macros in Rust: A Practical Introduction

Procedural macros in Rust are a powerful metaprogramming feature that allow you to write code that generates code at compile time. This enables you to reduce boilerplate, create domain-specific languages (DSLs), and extend Rust's syntax in novel ways. Unlike declarative macros (`macro_rules!`), which operate on pattern matching, procedural macros are full-fledged Rust functions that take a stream of tokens as input and produce a new stream of tokens as output.

There are three types of procedural macros:

  * **Function-like macros:** These are invoked with a `!` and can be used in expression or item positions.
  * **Derive macros:** These are used with the `#[derive]` attribute on structs and enums to automatically implement traits.
  * **Attribute macros:** These are custom attributes that can be attached to any item and can modify the item's definition.

### Setting Up a Procedural Macro Crate

To create a procedural macro, you must place it in a special "proc-macro" crate. This is a library crate with a specific configuration in its `Cargo.toml` file.

In your `Cargo.toml`, add the following:

```toml
[lib]
proc-macro = true

[dependencies]
syn = { version = "2.0", features = ["full"] }
quote = "1.0"
```

Here's what these dependencies are for:

  * **`proc-macro`**: This is a built-in Rust crate that provides the necessary types and functions for creating procedural macros.
  * **`syn`**: This crate is used for parsing Rust code from a `TokenStream` into a structured representation (an Abstract Syntax Tree or AST).
  * **`quote`**: This crate provides a quasi-quoting mechanism to turn Rust syntax trees back into `TokenStream`s.

-----

### Function-Like Macros

Function-like macros are the most general type of procedural macro. They take a `TokenStream` as input, which is the code written inside the macro invocation's parentheses, and they return a `TokenStream` that replaces the macro call.

Let's create a simple `sql!` macro that takes a string literal and prints it.

In your `lib.rs`:

```rust
use proc_macro::TokenStream;
use quote::quote;
use syn::{parse_macro_input, LitStr};

#[proc_macro]
pub fn sql(input: TokenStream) -> TokenStream {
    let input = parse_macro_input!(input as LitStr);
    let sql_query = input.value();

    let expanded = quote! {
        println!("Executing SQL: {}", #sql_query);
    };

    TokenStream::from(expanded)
}
```

**How to use it:**

In a separate crate that depends on your proc-macro crate, you can use it like this:

```rust
use your_proc_macro_crate::sql;

fn main() {
    sql!("SELECT * FROM users;");
}
```

When you run this code, it will print: `Executing SQL: SELECT * FROM users;`

-----

### Derive Macros

Derive macros are used to automatically implement traits for structs and enums. When you use `#[derive(MyTrait)]`, the derive macro for `MyTrait` is invoked with the `TokenStream` of the struct or enum it's attached to.

Let's create a derive macro for a simple `Answer` trait that returns the number 42.

First, define the trait in your main crate (or a shared core crate):

```rust
pub trait Answer {
    fn answer() -> i32;
}
```

Now, in your proc-macro crate's `lib.rs`:

```rust
use proc_macro::TokenStream;
use quote::quote;
use syn::{parse_macro_input, DeriveInput};

#[proc_macro_derive(Answer)]
pub fn answer_derive(input: TokenStream) -> TokenStream {
    let input = parse_macro_input!(input as DeriveInput);
    let name = input.ident;

    let expanded = quote! {
        impl Answer for #name {
            fn answer() -> i32 {
                42
            }
        }
    };

    TokenStream::from(expanded)
}
```

**How to use it:**

```rust
use your_proc_macro_crate::Answer;
// Assuming the Answer trait is in scope
// use your_main_crate::Answer;

#[derive(Answer)]
struct MyStruct;

fn main() {
    println!("The answer is: {}", MyStruct::answer());
}
```

This will print: `The answer is: 42`

-----

### Attribute Macros

Attribute macros allow you to create new attributes that can be attached to items like functions, structs, and enums. They receive two `TokenStream`s: one for the attribute itself (the part in parentheses) and one for the item it's attached to.

Let's create an `#[route(GET, "/")]` attribute that you might see in a web framework. This macro won't actually start a web server but will demonstrate how to parse the attribute's arguments and the function it's attached to.

In your `lib.rs`:

```rust
use proc_macro::TokenStream;
use quote::quote;
use syn::{parse_macro_input, AttributeArgs, ItemFn, NestedMeta};

#[proc_macro_attribute]
pub fn route(attr: TokenStream, item: TokenStream) -> TokenStream {
    let args = parse_macro_input!(attr as AttributeArgs);
    let func = parse_macro_input!(item as ItemFn);

    let method = if let Some(NestedMeta::Meta(syn::Meta::Path(path))) = args.get(0) {
        path.get_ident().unwrap()
    } else {
        panic!("Expected a method identifier");
    };

    let path = if let Some(NestedMeta::Lit(syn::Lit::Str(lit))) = args.get(1) {
        lit
    } else {
        panic!("Expected a string literal for the path");
    };

    let func_name = &func.sig.ident;

    let expanded = quote! {
        #func

        fn main() {
            println!("Route for function `{}` is: {} {}", stringify!(#func_name), stringify!(#method), #path);
        }
    };

    TokenStream::from(expanded)
}
```

**How to use it:**

```rust
use your_proc_macro_crate::route;

#[route(GET, "/")]
fn index() {
    // some logic
}
```

When you run this, it will print: ` Route for function  `index`  is: GET / `

This introduction covers the basics of writing each type of procedural macro in Rust. As you delve deeper, you'll discover more advanced techniques for parsing complex inputs, generating more sophisticated code, and providing excellent error messages to the users of your macros.