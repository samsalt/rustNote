# Storing UTF-8 Encoded Text with Strings in Rust

This note summarizes the key concepts of handling strings in Rust, based on Chapter 8.2 of the Rust Programming Language book.

## 1\. What is a String?

Rust has two main string types:

  * `&str`: Known as a "string slice," it's a borrowed, immutable reference to UTF-8 encoded string data stored elsewhere (e.g., in the program's binary).
  * `String`: A growable, mutable, owned, UTF-8 encoded string type provided by the standard library. It is essentially a wrapper around a vector of bytes (`Vec<u8>`).

When Rust developers refer to "strings," they could be talking about either `String` or `&str`. Both are UTF-8 encoded.

## 2\. Creating a String

You can create a `String` in several ways:

  * **Create an empty string:**

    ```rust
    let mut s = String::new();
    ```

  * **From a string literal (`&str`) using `.to_string()`:**

    ```rust
    let s = "initial contents".to_string();
    ```

  * **From a string literal using `String::from()`:**

    ```rust
    let s = String::from("initial contents");
    ```

    > `to_string()` and `String::from()` are equivalent; the choice is a matter of style.

  * **Strings can store any valid UTF-8 data:**

    ```rust
    let hello = String::from("Здравствуйте");
    ```

## 3\. Updating a String

### Appending

  * **`push_str`**: Appends a string slice (`&str`). This method does not take ownership of the parameter.
    ```rust
    let mut s1 = String::from("foo");
    let s2 = "bar";
    s1.push_str(s2); // s1 is now "foobar"
    println!("s2 is still available: {s2}"); // s2 can still be used
    ```
  * **`push`**: Appends a single character (`char`).
    ```rust
    let mut s = String::from("lo");
    s.push('l'); // s is now "lol"
    ```

### Concatenation

  * **Using the `+` Operator**:
    Combines two strings, but with a catch regarding ownership.

    ```rust
    let s1 = String::from("Hello, ");
    let s2 = String::from("world!");
    let s3 = s1 + &s2; // s1 is moved and can no longer be used
    ```

    The `add` method signature is `fn add(self, s: &str) -> String`. This means `s1` is moved (consumed), while `s2` is borrowed (`&s2` is coerced from `&String` to `&str`).

  * **Using the `format!` Macro**:
    A more readable and flexible way to combine strings, especially multiple ones. It does not take ownership of its parameters.

    ```rust
    let s1 = String::from("tic");
    let s2 = String::from("tac");
    let s3 = String::from("toe");
    let s = format!("{s1}-{s2}-{s3}"); // s is "tic-tac-toe"
    ```

## 4\. Indexing into Strings

**Direct indexing is not supported in Rust.** The following code will not compile:

```rust
// This code fails to compile!
let s1 = String::from("hi");
let h = s1[0];
```

### Why No Indexing?

1.  **Internal Representation**: A `String` is a wrapper over `Vec<u8>`.
2.  **Variable Byte Length**: UTF-8 characters can be 1 to 4 bytes long. For example, "Здравствуйте" is 12 characters but takes 24 bytes. An index into the bytes (`u8`) might not align with a character boundary.
3.  **Ambiguity**: It's not clear what an index should return: a byte, a character, or a grapheme cluster. Rust avoids this ambiguity to prevent bugs.
4.  **Performance**: Indexing is expected to be an O(1) operation, but with variable-length characters, Rust would have to iterate from the start to find the Nth character, which is an O(N) operation.

## 5\. Slicing Strings

While you can't index a single character, you can create a string slice using a range of bytes.

```rust
let hello = "Здравствуйте";
let s = &hello[0..4]; // s will be "Зд"
```

> **Warning**: Slicing requires caution. If you slice in the middle of a multi-byte character, your program will panic at runtime.
> `&hello[0..1]; // This will panic!`

## 6\. Methods for Iterating Over Strings

The safest way to process string elements is to be explicit about what you want.

  * **`.chars()`**: Iterates over Unicode scalar values (`char`).
    ```rust
    for c in "Зд".chars() {
        println!("{c}");
    }
    // Output:
    // З
    // д
    ```
  * **`.bytes()`**: Iterates over the raw bytes (`u8`).
    ```rust
    for b in "Зд".bytes() {
        println!("{b}");
    }
    // Output:
    // 208
    // 151
    // 208
    // 180
    ```

For more complex operations like iterating over grapheme clusters (what users perceive as "letters"), you should use external crates from [crates.io](https://crates.io/).