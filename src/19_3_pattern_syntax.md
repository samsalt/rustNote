# Pattern Syntax in Rust

Patterns are a special syntax in Rust for matching against the structure of types, both complex and simple. Using patterns allows you to destructure values, making it easier to work with their constituent parts.

### Matching Literals

You can match against literal values directly. This is useful for taking specific actions based on concrete values.

```rust
let x = 1;

match x {
    1 => println!("one"),
    2 => println!("two"),
    3 => println!("three"),
    _ => println!("anything"),
}
```

### Matching Named Variables

Named variables are patterns that match any value, binding it to the variable name. Be aware of shadowing when used inside a `match` expression, as a new variable is created within that scope.

```rust
let x = Some(5);
let y = 10;

match x {
    Some(50) => println!("Got 50"),
    Some(y) => println!("Matched, y = {y}"), // This `y` is new and shadows the outer `y`.
    _ => println!("Default case, x = {x:?}"),
}

println!("at the end: x = {x:?}, y = {y}");
// Matched, y = 5
// at the end: x = Some(5), y = 10
```

### Multiple Patterns

The `|` syntax acts as a pattern "or" operator, allowing you to match against multiple patterns in a single arm.

```rust
let x = 1;

match x {
    1 | 2 => println!("one or two"),
    3 => println!("three"),
    _ => println!("anything"),
}
```

### Matching Ranges with `..=`

The `..=` syntax allows matching an inclusive range of `char` or numeric values.

```rust
let x = 5;

match x {
    1..=5 => println!("one through five"),
    _ => println!("something else"),
}

let c = 'c';

match c {
    'a'..='j' => println!("early ASCII letter"),
    'k'..='z' => println!("late ASCII letter"),
    _ => println!("something else"),
}
```

## Destructuring

Patterns can be used to break apart structs, enums, and tuples to use their individual parts.

#### Destructuring Structs

You can break a struct into its fields. Rust offers a convenient shorthand where the pattern variable names match the field names.

```rust
struct Point {
    x: i32,
    y: i32,
}

let p = Point { x: 0, y: 7 };

// Long form
let Point { x: a, y: b } = p;
assert_eq!(0, a);
assert_eq!(7, b);

// Shorthand
let Point { x, y } = p;
assert_eq!(0, x);
assert_eq!(7, y);

// Destructuring with literal matching
match p {
    Point { x, y: 0 } => println!("On the x axis at {x}"),
    Point { x: 0, y } => println!("On the y axis at {y}"),
    Point { x, y } => println!("On neither axis: ({x}, {y})"),
}
```

#### Destructuring Enums

Destructuring an enum corresponds to the way data is defined within each variant.

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

let msg = Message::ChangeColor(0, 160, 255);

match msg {
    Message::Quit => { /* ... */ },
    Message::Move { x, y } => { /* ... */ },
    Message::Write(text) => { /* ... */ },
    Message::ChangeColor(r, g, b) => { /* ... */ },
}
```

## Ignoring Values in a Pattern

Sometimes you need to ignore parts or all of a value.

  * `_`: Ignores an entire value without binding it. This does not take ownership.
  * **Nested `_`**: Ignores a specific part of a larger value.
  * `_name`: Creates a variable but prevents compiler warnings for it being unused. **Note:** This *does* bind the value and takes ownership.
  * `..`: Ignores all remaining parts of a value, for example in a struct or tuple. It must be unambiguous.

<!-- end list -->

```rust
// Ignoring a function parameter
fn foo(_: i32, y: i32) {
    println!("This code only uses the y parameter: {y}");
}

// Ignoring parts of a tuple
let numbers = (2, 4, 8, 16, 32);
match numbers {
    (first, _, third, _, fifth) => { /* ... */ }
}

// Ignoring remaining parts of a struct
struct Point { x: i32, y: i32, z: i32 }
let origin = Point { x: 0, y: 0, z: 0 };
match origin {
    Point { x, .. } => println!("x is {x}"),
}

// Ignoring middle parts of a tuple
match numbers {
    (first, .., last) => {
        println!("Some numbers: {first}, {last}");
    }
}
```

## Advanced Pattern Matching

#### Extra Conditionals with Match Guards

A match guard is an additional `if` condition on a `match` arm. The arm is only chosen if both the pattern and the guard's condition are met.

```rust
let num = Some(4);

match num {
    Some(x) if x % 2 == 0 => println!("The number {x} is even"),
    Some(x) => println!("The number {x} is odd"),
    None => (),
}
```

#### `@` Bindings

The at operator (`@`) lets you create a variable that holds a value while simultaneously testing that value against a pattern.

```rust
enum Message {
    Hello { id: i32 },
}

let msg = Message::Hello { id: 5 };

match msg {
    Message::Hello { id: id_variable @ 3..=7 } => {
        println!("Found an id in range: {id_variable}");
    }
    Message::Hello { id: 10..=12 } => {
        println!("Found an id in another range");
    }
    Message::Hello { id } => {
        println!("Found some other id: {id}");
    }
}
```