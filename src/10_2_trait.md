# Rust Traits: Defining Shared Behavior

A **trait** in Rust defines shared functionality that a type can have. It's a way to group method signatures to define a set of behaviors. Traits are similar to what other languages call *interfaces*.

## Defining a Trait

You can define a trait using the `trait` keyword. Inside the trait, you declare the method signatures that describe the behaviors.

```rust
pub trait Summary {
    fn summarize(&self) -> String;
}
```

Any type that implements the `Summary` trait must provide its own implementation for the `summarize` method.

## Implementing a Trait on a Type

To implement a trait for a type, use the `impl TraitName for TypeName` syntax.

```rust
// A struct for news articles
pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

// Implementing the Summary trait for NewsArticle
impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}

// A struct for social media posts
pub struct SocialPost {
    pub username: String,
    pub content: String,
    pub reply: bool,
    pub repost: bool,
}

// Implementing the Summary trait for SocialPost
impl Summary for SocialPost {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}
```

**Orphan Rule**: You can implement a trait on a type only if the trait or the type (or both) are local to your crate. This is known as the *orphan rule* and it ensures coherence, preventing external crates from breaking your code.

## Default Implementations

Traits can have default implementations for their methods. This allows you to provide a fallback behavior that can be used or overridden by implementing types.

```rust
pub trait Summary {
    fn summarize(&self) -> String {
        String::from("(Read more...)") // Default implementation
    }
}

// This type will use the default implementation
impl Summary for NewsArticle {}
```

Default implementations can also call other methods within the same trait.

## Traits as Parameters

You can use traits to define functions that accept different types, as long as those types implement the specified trait.

### `impl Trait` Syntax

This is a concise way to specify that a parameter is of any type that implements a given trait.

```rust
pub fn notify(item: &impl Summary) {
    println!("Breaking news! {}", item.summarize());
}
```

### Trait Bound Syntax

This is a more verbose but more flexible syntax.

```rust
pub fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}
```

The trait bound syntax is necessary when you want to enforce that multiple generic parameters have the same type.

### Specifying Multiple Trait Bounds

Use the `+` syntax to require a type to implement multiple traits.

```rust
// Using impl Trait
pub fn notify(item: &(impl Summary + Display)) { /* ... */ }

// Using Trait Bounds
pub fn notify<T: Summary + Display>(item: &T) { /* ... */ }
```

### `where` Clauses for Clearer Bounds

For functions with many generic types and bounds, a `where` clause can improve readability.

```rust
fn some_function<T, U>(t: &T, u: &U) -> i32
where
    T: Display + Clone,
    U: Clone + Debug,
{
    // ...
}
```

## Returning Types That Implement Traits

You can use `impl Trait` in the return position to specify that a function returns some type that implements a trait, without naming the concrete type.

```rust
fn returns_summarizable() -> impl Summary {
    SocialPost {
        username: String::from("horse_ebooks"),
        content: String::from("of course, as you probably already know, people"),
        reply: false,
        repost: false,
    }
}
```

**Limitation**: The function must return a single, concrete type. Returning one of two different types that both implement the trait is not allowed with this syntax.

## Using Trait Bounds to Conditionally Implement Methods

You can use trait bounds on an `impl` block to conditionally implement methods or even entire traits for a generic type.

### Conditional Methods

```rust
use std::fmt::Display;

struct Pair<T> {
    x: T,
    y: T,
}

impl<T> Pair<T> {
    fn new(x: T, y: T) -> Self {
        Self { x, y }
    }
}

// This impl block only applies if T implements Display and PartialOrd
impl<T: Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!("The largest member is x = {}", self.x);
        } else {
            println!("The largest member is y = {}", self.y);
        }
    }
}
```

### Blanket Implementations

You can implement a trait for any type that satisfies certain bounds. This is common in the standard library. For example, `ToString` is implemented for any type that implements `Display`.

```rust
impl<T: Display> ToString for T {
    // ...
}
```