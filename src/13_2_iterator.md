# Understanding Iterators in Rust

Iterators provide a powerful and efficient way to process a sequence of items. They handle the underlying logic of iteration, allowing for cleaner and more flexible code.

## Core Concepts

### 1\. Laziness

In Rust, iterators are **lazy**. This means they have no effect until you call a method that consumes them.

*Creating an iterator by itself does nothing:*

```rust
let v1 = vec![1, 2, 3];
// This line alone doesn't perform any iteration.
let v1_iter = v1.iter();
```

### 2\. The `Iterator` Trait

All iterators implement the `Iterator` trait, which is central to their functionality.

  - **Required Method:** The only method you must implement is `next()`.
  - **Associated Type:** The trait defines an associated type, `Item`, which is the type the iterator returns.

<!-- end list -->

```rust
pub trait Iterator {
    type Item;
    fn next(&mut self) -> Option<Self::Item>;
    // other methods with default implementations...
}
```

The `next()` method returns each item wrapped in `Some`, and returns `None` when the sequence is finished. Calling `next()` advances the iterator, which is why the iterator variable must be mutable.

```rust
#[test]
fn iterator_demonstration() {
    let v1 = vec![1, 2, 3];
    let mut v1_iter = v1.iter(); // must be mutable

    assert_eq!(v1_iter.next(), Some(&1));
    assert_eq!(v1_iter.next(), Some(&2));
    assert_eq!(v1_iter.next(), Some(&3));
    assert_eq!(v1_iter.next(), None);
}
```

## Types of Iterator Methods

There are two main categories of methods for iterators.

### 1\. Consuming Adapters

These methods call `next()` on the iterator until it's exhausted and are used to get a final result. Once called, the iterator can no longer be used.

  - **Example:** The `sum()` method takes ownership of the iterator, iterates through all items, and returns their sum.

<!-- end list -->

```rust
#[test]
fn iterator_sum() {
    let v1 = vec![1, 2, 3];
    let v1_iter = v1.iter();

    // v1_iter is consumed here
    let total: i32 = v1_iter.sum();

    assert_eq!(total, 6);
}
```

### 2\. Iterator Adapters

These methods transform an iterator into a new iterator. They are lazy and don't do any work until the new iterator is consumed. This allows for chaining multiple transformations together.

  - **Example:** The `map()` method takes a closure and applies it to each element, producing a new iterator with the modified elements.

This code produces a warning because the new iterator is never used:

```rust
let v1: Vec<i32> = vec![1, 2, 3];

// Warning: iterators are lazy and do nothing unless consumed
v1.iter().map(|x| x + 1);
```

To use the result, you must chain a consuming adapter, like `collect()`, which gathers the results into a new collection.

```rust
let v1: Vec<i32> = vec![1, 2, 3];

// `collect()` consumes the iterator and creates a new vector
let v2: Vec<_> = v1.iter().map(|x| x + 1).collect();

assert_eq!(v2, vec![2, 3, 4]);
```

## Using Closures with Iterators

Iterator adapters often use closures, which can capture variables from their environment. This makes them incredibly flexible.

  - **Example:** The `filter()` method uses a closure that returns a boolean. The new iterator will only yield items for which the closure returns `true`.

The following function filters a vector of `Shoe` structs based on `shoe_size` captured from its environment.

```rust
#[derive(PartialEq, Debug)]
struct Shoe {
    size: u32,
    style: String,
}

fn shoes_in_size(shoes: Vec<Shoe>, shoe_size: u32) -> Vec<Shoe> {
    shoes.into_iter()
         // The closure captures `shoe_size` from the function's scope
         .filter(|s| s.size == shoe_size)
         .collect()
}
```