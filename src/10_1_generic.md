# **Generic Data Types in Rust**

Generics are a tool for creating definitions for items like functions and structs that can be used with various concrete data types. This avoids code duplication and increases flexibility.

-----

### **1. In Function Definitions**

Generic type parameters are declared in a function's signature to create a single function that can operate on different data types.

  * **Syntax**: Declare the generic type inside angle brackets `<>` between the function name and the parameter list. The convention is to use `T` for "type".

    ```rust
    fn largest<T>(list: &[T]) -> &T {
      // ...
    }
    ```

  * **Traits**: To use operations like comparison (`>`), the generic type must be constrained to types that implement the necessary trait (e.g., `std::cmp::PartialOrd`).

    ```rust
    // This function requires T to be a type that can be ordered.
    fn largest<T: std::cmp::PartialOrd>(list: &[T]) -> &T {
      // ...
    }
    ```

-----

### **2. In Struct Definitions**

Structs can use generic types for one or more of their fields.

  * **Single Generic Type**: Both fields `x` and `y` must be of the same type `T`.

    ```rust
    struct Point<T> {
        x: T,
        y: T,
    }

    // `integer` has x and y as i32
    let integer = Point { x: 5, y: 10 };
    // `float` has x and y as f64
    let float = Point { x: 1.0, y: 4.0 };
    ```

  * **Multiple Generic Types**: Fields can have different types by using multiple generic parameters.

    ```rust
    struct Point<T, U> {
        x: T,
        y: U,
    }

    // This is valid because T can be an integer and U a float
    let integer_and_float = Point { x: 5, y: 4.0 };
    ```

-----

### **3. In Enum Definitions**

Enums can be defined to hold generic data types in their variants.

  * **`Option<T>`**: An enum that is generic over type `T`. It represents an optional value.

    ```rust
    enum Option<T> {
        Some(T),
        None,
    }
    ```

  * **`Result<T, E>`**: An enum generic over two types: `T` for success and `E` for an error.

    ```rust
    enum Result<T, E> {
        Ok(T),
        Err(E),
    }
    ```

-----

### **4. In Method Definitions**

Methods can be implemented on generic structs and enums.

  * **Generic `impl` Block**: The `impl` keyword must also declare the generic type `T` to specify you're implementing methods on the generic type `Point<T>`.

    ```rust
    impl<T> Point<T> {
        fn x(&self) -> &T {
            &self.x
        }
    }
    ```

  * **Concrete `impl` Block**: You can also implement methods for only a specific concrete type. This block does not declare a generic type after `impl`.

    ```rust
    // This method is only available on Point<f32> instances.
    impl Point<f32> {
        fn distance_from_origin(&self) -> f32 {
            (self.x.powi(2) + self.y.powi(2)).sqrt()
        }
    }
    ```

-----

### **5. Performance of Generics**

There is **no runtime cost** for using generics in Rust.

  * **Monomorphization**: Rust turns generic code into specific code at compile time. It creates a specialized version of the generic code for each concrete type used.
  * **Result**: The compiled code runs just as efficiently as if you had written separate, specific definitions for each type by hand.