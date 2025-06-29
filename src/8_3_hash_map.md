# Rust Notes: Hash Maps

The `HashMap<K, V>` type stores a mapping of keys of type `K` to values of type `V`. It uses a hashing function to determine how to store the data in memory.

## Creating a New Hash Map

You must first `use` the `HashMap` from the standard library's collections. Hash maps store their data on the heap.

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);
```

**Key Points:**

  * All keys must have the same type.
  * All values must have the same type.

## Accessing Values in a Hash Map

Use the `get` method with a key to retrieve a value.

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

let team_name = String::from("Blue");
// get returns an Option<&V>, so we handle it.
let score = scores.get(&team_name).copied().unwrap_or(0); // score is 10
```

You can also iterate over the key-value pairs in a hash map. The order is arbitrary.

```rust
for (key, value) in &scores {
    println!("{key}: {value}");
}
// Example Output:
// Yellow: 50
// Blue: 10
```

## Hash Maps and Ownership

  * **For `Copy` types (e.g., `i32`)**: Values are copied into the hash map.
  * **For owned types (e.g., `String`)**: Values are moved into the hash map, and the hash map becomes the owner.

<!-- end list -->

```rust
let field_name = String::from("Favorite color");
let field_value = String::from("Blue");

let mut map = HashMap::new();
map.insert(field_name, field_value);
// field_name and field_value are now invalid as they've been moved.
```

## Updating a Hash Map

Each unique key can only have one value associated with it at a time.

### 1\. Overwriting a Value

If you `insert` a key that already exists, the associated value will be overwritten.

```rust
let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Blue"), 25); // Overwrites 10 with 25

println!("{:?}", scores); // Prints {"Blue": 25}
```

### 2\. Adding a Key Only If It Isn’t Present

The `entry` method checks for a key's existence and allows conditional insertion using `or_insert`.

  * If the key exists, `or_insert` returns a mutable reference to its value.
  * If the key does not exist, it inserts the new value and returns a mutable reference to it.

<!-- end list -->

```rust
let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);

// "Yellow" does not exist, so it's inserted.
scores.entry(String::from("Yellow")).or_insert(50);
// "Blue" already exists, so nothing changes.
scores.entry(String::from("Blue")).or_insert(50);

println!("{:?}", scores); // Prints {"Yellow": 50, "Blue": 10}
```

### 3\. Updating a Value Based on the Old Value

This is a common pattern for tasks like counting occurrences.

```rust
let text = "hello world wonderful world";
let mut map = HashMap::new();

for word in text.split_whitespace() {
    // Get a mutable reference to the word's count, or insert 0.
    let count = map.entry(word).or_insert(0);
    // Dereference the mutable reference and increment it.
    *count += 1;
}

println!("{:?}", map); // Prints {"world": 2, "hello": 1, "wonderful": 1}
```

## Hashing Functions

  * By default, `HashMap` uses the **SipHash** function, which offers good security against Denial-of-Service (DoS) attacks.
  * If performance profiling shows the hash function is a bottleneck, you can specify a different hasher by using a type that implements the `BuildHasher` trait.