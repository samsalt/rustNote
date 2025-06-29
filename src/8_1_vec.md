# Storing Lists of Values with Vectors in Rust

Vectors (`Vec<T>`) are a fundamental collection type in Rust. They allow you to store a variable number of values of the same type in a contiguous block of memory.

### Creating a New Vector

There are two primary ways to create a vector:

1.  **`Vec::new()`**: Creates a new, empty vector. You must provide a type annotation if Rust can't infer the type of the elements.

    ```rust
    let v: Vec<i32> = Vec::new();
    ```

2.  **`vec!` macro**: A convenient macro to create a vector with initial values. Rust can typically infer the type from the provided values.

    ```rust
    let v = vec![1, 2, 3]; // Rust infers Vec<i32>
    ```

### Updating a Vector

To add elements to a vector, you must first declare it as mutable (`mut`). You can then use the `push` method.

```rust
let mut v = Vec::new();
v.push(5);
v.push(6);
```

### Reading Elements from a Vector

You can access elements in a vector in two ways:

1.  **Indexing (`[]`)**:

      * Returns a reference (`&`) to the element at the given index.
      * The program will **panic** if you try to access an index that is out of bounds.

    <!-- end list -->

    ```rust
    let v = vec![1, 2, 3, 4, 5];
    let third: &i32 = &v[2];
    // let does_not_exist = &v[100]; // This would panic!
    ```

2.  **`get()` method**:

      * Returns an `Option<&T>`.
      * It returns `Some(&element)` if the index is in bounds, and `None` if it is out of bounds, without panicking. This is useful for handling invalid access gracefully.

    <!-- end list -->

    ```rust
    let v = vec![1, 2, 3, 4, 5];
    match v.get(2) {
        Some(third) => println!("The third element is {third}"),
        None => println!("There is no third element."),
    }
    // let does_not_exist = v.get(100); // Returns None
    ```

**Ownership and Borrowing Rules:**
You cannot have both mutable and immutable references to a vector in the same scope. This is because adding a new element (`v.push()`) might require reallocating the vector's memory, which would invalidate any existing references to its elements.

### Iterating Over a Vector

You can loop over the elements of a vector using a `for` loop.

  * **Immutable iteration**:

    ```rust
    let v = vec![100, 32, 57];
    for i in &v {
        println!("{i}");
    }
    ```

  * **Mutable iteration**: To modify elements, iterate over mutable references. You need to use the dereference operator (`*`) to change the value.

    ```rust
    let mut v = vec![100, 32, 57];
    for i in &mut v {
        *i += 50;
    }
    ```

### Storing Multiple Types with Enums

Vectors can only hold elements of the same type. To store elements of different types in a single vector, you can define an `enum` and store the enum variants in the vector.

```rust
enum SpreadsheetCell {
    Int(i32),
    Float(f64),
    Text(String),
}

let row = vec![
    SpreadsheetCell::Int(3),
    SpreadsheetCell::Text(String::from("blue")),
    SpreadsheetCell::Float(10.12),
];
```

### Dropping a Vector

Like any other struct, a vector is dropped when it goes out of scope. When the vector is dropped, all of its contents are also dropped, and the memory they used is freed.

```rust
{
    let v = vec![1, 2, 3, 4];
    // do stuff with v
} // <- v goes out of scope and is freed here
```