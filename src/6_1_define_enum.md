### Enums in Rust

Enums (enumerations) allow you to define a type by enumerating its possible variants. A value can be one of a possible set of values.

-----

#### 1\. Defining an Enum

You can define an enum by listing its possible variants.

**Example: IP Address Kind**

```rust
enum IpAddrKind {
    V4,
    V6,
}
```

You can then create instances of these variants, which are namespaced under the enum's identifier:

```rust
let four = IpAddrKind::V4;
let six = IpAddrKind::V6;

fn route(ip_kind: IpAddrKind) {}

route(IpAddrKind::V4);
route(IpAddrKind::V6);
```

-----

#### 2\. Storing Data with Enum Variants

Unlike a struct, an enum allows you to put data directly into each variant. This makes the enum a single, more concise type.

  * Each variant can have different types and amounts of associated data.
  * The name of each enum variant also becomes a constructor function for that variant.

**Example: Storing IP Address Data**

```rust
enum IpAddr {
    V4(u8, u8, u8, u8),
    V6(String),
}

let home = IpAddr::V4(127, 0, 0, 1);
let loopback = IpAddr::V6(String::from("::1"));
```

**Example: Diverse Data in Variants**

```rust
enum Message {
    Quit, // No data associated
    Move { x: i32, y: i32 }, // Anonymous struct
    Write(String), // A single string
    ChangeColor(i32, i32, i32), // A tuple of three i32s
}
```

-----

#### 3\. Methods on Enums

Similar to structs, you can define methods on enums using an `impl` block. The method can use `self` to access the value of the instance it's called on.

```rust
impl Message {
    fn call(&self) {
        // Method body would be defined here
    }
}

let m = Message::Write(String::from("hello"));
m.call();
```

-----

#### 4\. The `Option` Enum: An Alternative to Null

Rust does not have `null`. Instead, it uses the `Option<T>` enum to handle cases where a value could be something or nothing.

The `Option<T>` enum is defined as:

```rust
enum Option<T> {
    None,
    Some(T),
}
```

  * It is included in the prelude, so you can use `Some` and `None` directly.
  * The generic type parameter `<T>` means the `Some` variant can hold data of any type.
  * This approach avoids bugs common with `null` values because the compiler forces you to handle the `None` case. You must convert an `Option<T>` to a `T` before you can perform operations on it.

**Example Usage:**

```rust
let some_number = Some(5);
let some_char = Some('e');

let absent_number: Option<i32> = None;
```

To use an `Option<T>` value, you must have code that handles both the `Some(T)` and `None` variants. The `match` expression is a common way to do this.