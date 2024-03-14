# Brief Intro To Rust

Disclaimer, this not workshop, it's just a quick overview about Rust and its unique concepts and 
features.

---


## What's Rust?

Rust is general-purpose programming language designed for performance and 
safety, especially safe concurrency.

---

## Rust history

- **2006**: The language grew out of personal project by Graydon Hoare
- **2009**: Mozilla began sponsoring the project
- **2010**: Mozilla announced the language Rust
- **2015**: First stable release, was released (Rust 1.0)
- **2021**: Rust foundation was officially announced

---

## Why Rust is interesting?

===

## Safety

Rust’s rich type system and ownership model guarantee memory-safety and thread-safety — enabling you
to eliminate many classes of bugs at compile-time.

===

#### Bugs that cannot exist in Rust

- [Data-race](https://en.wikipedia.org/wiki/Race_condition#Data_race) (race condition)
- [Out-of bound access](https://en.wikipedia.org/wiki/Buffer_over-read)  (with arrays)
- [Null pointer](https://en.wikipedia.org/wiki/Null_pointer) (There is no `null` value in Rust)
- [Uninitialized variable](https://en.wikipedia.org/wiki/Uninitialized_variable)
- [Dangle pointers](https://en.wikipedia.org/wiki/Dangling_pointer)
- Use after free
- Double free

Rust compiler will catch all these bugs at compile-time

===

### Are These BUGs Really Harmful?

Well, big tech companies think so.. saying:
- **Microsoft** says ~70% of the vulnerabilities each year continue to be memory safety issues and
  believes that Rust is the solution
  ([source](https://msrc-blog.microsoft.com/2019/07/16/a-proactive-approach-to-more-secure-code/))
- **Google** saying the same and supporting Rust officially for developing
  Android OS itself.
  ([source](https://security.googleblog.com/2021/04/rust-in-android-platform.html))
- **Linux Kernel**: In the process to make Rust second class language for the Kernel
  development. ([source](https://thenewstack.io/rust-in-the-linux-kernel-good-enough/))

===

## Performance

Rust is blazingly fast and memory-efficient: with no runtime or garbage collector, it can power
performance-critical services, run on embedded devices, and easily integrate with other languages.

===

## Productivity

Rust has great documentation, a friendly compiler with useful error messages, and top-notch tooling
— an integrated package manager and build tool, smart multi-editor support with auto-completion and
type inspections, an auto-formatter, and more.


---

# Rust By Example

===

## Hello World

```rust [2,4-5|1-2,5|3-4]
// This is the main function
fn main() {
  // Print text to the console
  println!("Hi there o/ "); 
}
```
note: println! is a macro that prints text to the console.

===

## Rust Core Concepts

Rust has 3 unique concepts called **Ownership**,
**Borrow** and **Lifetime** which ensures your code is safe.

===

### Ownership

- Ownership Rules:
  - Each **value** has a **variable** that’s called its **owner**.
  - There can only be one owner at a time.
  - When the owner goes out of scope, the value will be dropped.

===

### Ownership in action
```rust [2-3|2-8|2-11]
fn main() {
  // `name` is a variable, and it's the owner of `Max` value
  let name = String::from("Max");
  
  // *Move* the value ownership from `name` into `x`
  // now `x` is the new owner, and `name`
  // doesn't own a value anymore
  let x = name;
  
  // `x` = Max
  // `name` cannot be used, because it' doesn't own value 
}
```

===

There is an exception. Types implement `Copy` trait (interface) will get copied instead of 
moving them. 

Primitive types is an example
```rust [2-3|2-6|2-8]
fn main() {
  // `x` is a variable, and it's the owner of `25` value
  let x = 25;
  
  // *Copy* `x` into `y`
  let y = x;
  
  // x = 25, and y = 25
}
```

===

#### What if we want to reuse data?

```rust [5-7|1-6|1-10|1-14]
fn greeting(username: String) {
  println!("Hey {username}. How are you doing today?");
}

fn main() {
  let name = String::from("Max");
  
  // *Move* the value from `name` into function `greeting`.
  greeting(name);
  
  // Now I cannot use `name` anymore!
  greeting(name); // <--- Error: Compiler will yell at you!
                  //      name doesn't have value anymore
}
```

===

### Borrow & Lifetime 

- Borrow & Lifetime Rules:
  - At any given time, you can have either one mutable reference or any number of immutable
    references.
  - References must always be valid.

===

#### Borrowing with Immutable Reference

```rust [5-6,13|1-7,13|1-10,13|1-13]
fn greeting(username: &String) {
  println!("Hey {username}. How are you doing today?");
}

fn main() {
  let name = String::from("Max");
  
  // *Borrow* the value from `name`
  greeting(&name);
  
  // Now we can use `name` again
  greeting(&name); // <--- Compiler is happy now
}
```

===

#### Borrowing with Mutable Reference

```rust [5-6,13|1-6,13|1-10,13|]
fn add_greeting(greeting: &mut String) {
  greeting = format!("Hi {}, what's up?");
}

fn main() {
  let mut name = String::from("Max");
  
  // *Borrow* the value from `name`, and the borrower
  // can write on the value because we use `&mut`
  greeting(&mut name);
  
  // name = Hi Max, what's up?
}
```

===

### Lifetime 

- Lifetimes are what the Rust compiler uses to keep track of how long references are valid for. 
- Checking references is one of the borrow checker’s main responsibilities.
- Lifetimes help the borrow checker ensure that you never have invalid references.

```rust
foo<'a>
// `foo` has a lifetime parameter `'a`
```

===

### Lifetime by Example 

```rust [1-6,13|7-11,13] 
//trims leading and trailing "x" characters from a string 
//and returns a reference to the trimmed substring
fn trim_x(val: &String) -> &str {
    val.trim_matches('x')
}

fn main() {
    let val: String = "xxhelloxx".to_string();
    println!("val is '{}'", val);
    println!("trimmed is '{}'", trim_x(&val));
}
```
Output:
```
val is 'xxhelloxx'
trimmed is 'hello'
```

===


### Lifetime by Example 

<span style="font-size:0.7em;">

The function `trim_x` accepts a reference and returns a reference.

If this is valid code, then there are only 2 possibilities where that returned reference could point to: 
- Either it points to static memory (e.g. a constant) 
- Or it points to val's memory! 

If a function accepts a reference and returns a reference, then the two lifetimes must be the same.

This is what's actually happening behind the scenes:

</span>

```Rust 
fn trim_x<'a>(val: &'a String) -> &'a str {
    val.trim_matches('x')
}
```
<span style="font-size:0.7em;">

This defines a lifetime `'a`, and then says that `val` and the returned reference must have the same lifetime. 
However, for ergonomic reasons, as long as Rust can automatically derive the lifetimes, you don't have to write them. This is called "lifetime elision".

</span>

===

### Lifetime by Example 
But what happens when you do this?

```Rust
fn trim_x(val: &String, msg: &String) -> &str {
    println!("Calling trim_x with message {}", msg);
    val.trim_matches('x')
}
```

Now the function accepts two references and returns a single reference. But does the returned reference point into memory of the first or of the second parameter? Rust can't know!

===

### Lifetime by Example 
```Rust
error[E0106]: missing lifetime specifier
 --> src/main.rs:1:42
  |
1 | fn trim_x(val: &String, msg: &String) -> &str {
  |                                          ^ expected lifetime parameter
  |
  = help: this function's return type contains a borrowed value, but the signature does not say whether it is borrowed from `val` or `msg`
```

===

### Lifetime by Example 
We need to help the compiler a little.

```Rust

fn trim_x<'a>(val: &'a String, msg: &String) -> &'a str {
    println!("Calling trim_x with message {}", msg);
    val.trim_matches('x')
}

```

===

## Rust Types

Rust is a strongly typed language, thus we need to talk using types right?

===

### Primitive Types

Rust has the following primitive types:

<div style="font-size: 0.9em">

| type              | variant                                    |
|-------------------|--------------------------------------------|
| signed integers   | `i8`, `i16`, `i32`, `i64`, `i128`, `isize` |
| unsigned integers | `u8`, `u16`, `u32`, `u64`, `u128`, `usize` |
| floating point    | `f32`, `f64`                               |
| character         | `char` (Unicode)                           |
| boolean           | `bool`                                     |
| unit type         | `()`                                       |

</div>

===

### And we have these compound types
- arrays like `[1, 2, 3, 4]`
- tuples like `(1, true, "Max")`

===

### Custom Types
Rust support two custom types
- struct
- enum

```rust
// fields are private by default
struct User {
  username: String,
  state: State,
  pub birth_date: DateTime, // this field is public because
}                           // we use `pub` keyword

// enum in Rust is so POWERFUL, because we can store data
// inside the enum foreach variant!
enum State {
  Active, // variant without data
  Locked {
    reason: String, // `reason` associated with variant `Locked`
  }
}
```

===

### Where is class?

Rust doesn't have `class` keyword, neither it has the inheritance concept! 

People at Rust believes that inheritance makes the code hard to understand, do
refactor and troubleshooting of the code. So they remove it.

`struct`s can be used where class needed, Rust also provide you with alternative
concept, called `trait`, we will see that later on.

===

### So, Where is methods?

<div style="font-size: 0.9em">
Unlike other languages, methods is defined in separate `impl` block

```rust [1-5|7,22|8-16|18-21|1-22]
struct User {
  username: String,
  state: State,
  birth_date: DateTime,
}

impl User {
  // constructor
  fn new(username: String, birth_date: DateTime) -> Self {
    // this is how we construct `struct` types:
    User { 
      username,
      birth_date, 
      state: State::Active
    }
  }
  
  // method
  fn age(&self) -> i32 {
    todo!()
  }
}
```
</div>

===

## Closures

Rust support closures (lambda)
```rust
fn main() {
  // this is how we assign closure to a variable
  let add_two = |value| value + 2;
  
  // and we use it like a function
  let x = add_two(8);
  
  // x = 10
}
```

===

## Closures Can Capture Environment

We can capture environment variables
```rust
fn main() {
  let username = String::from("Max");

  // this is how we capture variables into closures
  let greatting = move || format!("Hi {username}") ;
  
  // and we use it like a function
  let x = greatting();
  
  // x = Hi Max
}
```

===

#### if/else

```rust [2-10|12-21]
fn main() {
  let n = 5;
  
  if n < 0 {
    println!("{n} is negative");
  } else if n > 0 {
    println!("{n} is positive");
  } else {
    println!("{0} is zero");
  }

  // we can also use `if` as value!
  let age = 38;
  
  let is_young = if age < 30 {
    true
  } else {
    false
  };
  
  // is_young = false
}
```

===

## Loops
Rust have 3 loops keyword, `loop`, `while`, `for` and they can be used in different ways.

===

### loop expression

```rust [2-4|6-13]
fn main() {
  loop {
    pritnln!("endless loop");
  }
  
  let mut counter = 0;
  let result = loop {
    counter += 1;
    
    if counter == 10 {
      break counter * 2;
    }
  };
}
```

===

### for expression

```rust [2-5|7-10]
fn main() {
  // iterating over range
  for i in 0..10 {
    pritnln!("current value is {i}")
  }

  // iterating over array
  for name in ["Max", "Yousef", "Saad"] {
    println!("current name is {name}");
  }
}
```

===

## Match pattern

Rust provides match pattern via the `match` keyword (similar to switch, put more powerful).

```rust
fn main() {
  let number = 13;

  match number {
    // Match a single value
    1 => println!("One!"),
    // Match several values
    2 | 3 | 5 | 7 | 11 => println!("This is a prime"),
    // Match an inclusive range
    13..=19 => println!("A teen"),
    // Handle the rest of cases
    _ => println!("Ain't special"),
  }
}
```

===

## Modules System

Rust provides a powerful module system that can be used to hierarchically split code in logical
units (modules), and manage visibility (public/private) between them.

===

### Module (simple example)

```rust [1-2,18|3-6|8-17|20-22|24-34]
// A module named `my_mod`
mod my_mod {
  // Items in modules default to private visibility.
  fn private_function() {
    todo!()
  }

  // Modules can also be nested
  pub mod nested {
    pub fn function() {
      todo!()
    }

    fn private_function() {
      todo!()
    }
  }
}

fn function() {
  todo!()
}

fn main() {
  // Modules allow disambiguation between items that have the same name.
  function();
  my_mod::nested::function();

  // Error! `private_function` is private
  //my_mod::private_function();

  // Error! `private_function` is private
  //my_mod::nested::private_function();
}
```

===

## Traits

A trait is a collection of methods defined for an unknown type: Self. They can access other methods
declared in the same trait.

===

```rust 
struct Point {
  x: i32,
  y: i32,
}

struct Rectangle {
  x: i32,
  y: i32,
  width: i32,
  height: i32,
}

pub trait Move {
  // required method
  fn move_to(&mut self, x: i32, y: i32);
  fn x(&self) -> i32;
  fn y(&self) -> i32;

  // provided method (have default implementation)
  fn move_by(&mut self, x: i32, y: i32) {
    self.move_to(self.x() + x, self.y() + y);
  }
}

impl Move for Point {
  fn move_to(&mut self, x: i32, y: i32) {
    self.x = x;
    self.y = y;
  }
  
  fn x(&self) -> i32 {
    self.x
  }
  
  fn y(&self) -> i32 {
    self.y
  }
}

impl Move for Rectangle {
  fn move_to(&mut self, x: i32, y: i32) {
    self.x = x;
    self.y = y;
  }
  
  fn x(&self) -> i32 {
    self.x
  }
  
  fn y(&self) -> i32 {
    self.y
  }
}

fn move_any_thing_impl_move(object: &mut impl Move) {
  object.move_by(5, 5)
}

```

===

## Generics<T>

Rust support generics, and gives you the ability to put some constraints on them!
```rust [1-3|5-11|13-19]
struct Container<T> {
  item: T,
}

// `T` is bound to impl `Display`
struct PrintableType<T> 
where 
  T: Display,
{
  item: T,
}
```

===

## Generics With Functions

```rust [1-6|8-11]
fn print_me<T>(value: T)
where
  T: Display,
{
  println!("{value}");
}

// another way to do it is using `impl`
fn print_me_impl(value: impl Display) {
  println!("{value}");
}
```

===

### Traits from `std` library

Rust standard library provide you very helpful traits that we can use and reuse in our code.

- **`From`**: Used to do construct type from another value
- **`Display`**: Used to make types printable
- **`Error`**: Used by error types
- **`Add`**: The addition operator +
- **`Clone`**: gives the ability to explicitly duplicate an object
- ..etc

===

### Generics & Trait impls

Using generics and traits we can reuse code very effectively

<div style="width: 120%; transform: translateX(-10%)">

```rust [1-3|1-14]
pub trait DisplayUppercase {
  fn uppercase(&self) -> String;
}

// With this impl, any type impl `Display` will 
// impl `DisplayUppercase` even types that's not written yet :)
impl<T> DisplayUppercase for T 
where 
  T: Display
{
  fn uppercase(&self) -> String {
    format!("{}", self.to_string().to_uppercase())
  }
}
```

</div>

===

## Errors

Unlike other common languages, Rust doesn't have exception concept, instead it
uses `Result` type for functions that may fail while doing its job.

<div style="width: 120%; transform: translateX(-10%)">

```rust [1-5|7-10|12-20|23-24|26-30|32-36|38-41]
// This is the Result type:
pub enum Result<T, E> {
  Ok(T),
  Err(E)
}

enum LoginError {
  InvalidCredentials,
  OtherError,
}

fn login(username: String, pwd: String) -> Result<SessionId, LoginError> {
  if username == "root" && pwd == "toor" {
    // login successfully
    Ok(SessionId::generate())
  } else {
    // login failed
    Err(LoginError::InvalidCredentials)
  }
}

fn main() {
  // try to login to the system
  let login_result = login("badboy", "12345");

  // handle both success & failure the result using match pattern
  match login_result {
    Ok(id) => println!("Logged in with session id {id}"),
    Err(err) => pritnln!("Invalid credentials"),
  }
  
  // get the success only result using if let pattern
  if let Ok(id) = login_result {
    // do things with the return session id
    println!("Logged in with session id {id}");
  }
  
  // get the success value using unwarp() method, this will panic (abort)
  // the application if the login_result was Err, which I don't recommend
  let id = login_result.unwarp();
  println!("Logged in with session id {id}");
}
```

</div>


===

### Unrecoverable Errors

Sometimes the application get into unrecoverable state (e.g. fatal errors), and
we have to terminate the application.
  
Rust have `panic!()` for this situation

```rust
fn main() {
  let bad_state = true;
  if bad_state {
    panic!("message to display before we exit the app");
  }

  println!("we all good to go!");
}
```

===

## Where is `null`?

Rust doesn't have `null` concept, variables must be initialized before we can use them.

How can we express that a variable might not have a value then?

===

## `Option` Type

Rust have `Option` type that replace `null` concept in other languages.

<div style="width: 120%; transform: translateX(-10%)">

```rust [1-5|7-13|15-17|15-23|25-30|31-35]
// Option type:
pub enum Option<T> {
  Some(T),
  None
}

fn try_division(dividend: i32, divisor: i32) -> Option<i32> {
  if divisor == 0 {
    None
  } else {
    Some(dividend / divisor)
  }
}

fn main() {
  let some_value = try_division(4, 2); // Some(2)
  let no_value = try_division(2, 0);   // None
  
  // extract value using match pattern
  match some_value {
    Some(value) => pritnln!("some_value: {value}"),
    None => pritnln!("some_value was: None"),
  };
  
  // or extract value using if let syntax 
  // if we only interested in the some value
  if let Some(value) = no_value {
    println!("no_value: {value}");
  }
  
  // or we can use unwarp() method to get the inner
  // value if any or panic and abort the application
  // (which I don't recommend :)
  let inner_value = some_value.unwarp(); // this not cool way don't do that at Home
  pritnln!("inner_value: {inner_value}");
}
```

</div>

===


## Macros

Rust provides a powerful macro system that allows meta-programming.

Rust have two type of macros:
- Macros By Example
- Procedural Macros

===

### So why macros useful?

1) Don't repeat yourself.
2) Domain-specific languages. Macros allow you to define special syntax for a specific purpose.
3) Variadic interfaces. Sometimes you want to define an interface that takes a variable number of
  arguments. An example is `println!` which could take any number of arguments, depending on the
  format string!

===

### How Macros Work In Rust?

Unlike C macros and other languages, Rust macros takes **Rust token** and expand
into abstract syntax trees (AST) or let's say **valid Rust code**, rather than
string preprocessing, so you don't get unexpected precedence bugs.

===

### DSL in macros

A DSL is a mini "language" embedded in a Rust macro. It is completely valid Rust because the macro
system expands into normal Rust constructs.

===

### Let's write some macros

```rust [1-8|10-20]
macro_rules! calculate {
    (eval $e:expr) => {{
        {
            let val: f32 = $e; // Force types to be float
            //println!("{} = {}", stringify!($e), val);
        }
    }};
}

fn main() {
    // will print: 1.0 + 2.0 = 3
    calculate! {
        eval 1.0 + 2.0 // hehehe `eval` is _not_ a Rust keyword!
    }

    // will print: (1.0 + 2.0) * (3.0 / 4.0) = 2.25
    calculate! {
        eval (1.0 + 2.0) * (3.0 / 4.0)
    }
    
    // So we have written eval expression in Rust macro_rules,
    // and it's compiled time checked!
}
```

===

### Procedural Macros

Rust also have another kind of macros that is more powerful called `Procedural Macro`.

In short, it allows you to analyze the passed code and generate code upon it.
It's like we write compiler to our own macro!!

===

### Macros from community?

Rust community have written many powerful macros:

<div class="r-stack" style="font-size: 0.9em; text-align:left; width: 100%">
  <div class="fragment fade-out">

- <!-- .element: class="fragment fade-in-fade-out" data-fragment-index="1" -->
  `Serialize`/`Deserialize` that makes your type de/serializable to any format
  including `json`, `yaml`.. etc
- <!-- .element: class="fragment" data-fragment-index="2" -->
  `TypedBuidler` macro add builder pattern to your structs with compile time check!!
- <!-- .element: class="fragment" data-fragment-index="3" --> 
  `Getter`/`Setter` adds setter and getter to your structs
- <!-- .element: class="fragment" data-fragment-index="4" --> 
  `json!()` macro that let your write raw JSON, with compile time check!.

  </div>

  <div class="fragment fade-in">

- <!-- .element: class="fragment" -->
  `html!()` JSX like macro, that let your build dynamic views
- <!-- .element: class="fragment" -->
  `query!()` macro that check and validate your raw SQL against real database
- <!-- .element: class="fragment" -->
  macros to embed `Python` and `Js` code inside Rust!
- <!-- .element: class="fragment" -->
  and many more

  </div>
</div>

===

### Serde example

<div style="width: 140%; transform: translateX(-15%)">

```rust [1-8|1-15|10-18|17-21|23-33]
#[derive(Serialize, Deserialize)]
pub struct User {
    #[serde(alias = "Username")]
    username: String,
    email: String,
    #[serde(skip_serializing_if = "Option::is_none")]
    phone: Option<String>,
}

fn main() {
  let user = User {
    username: "Max".to_string(),
    email: "Max@domain.com".to_string(),
    phone: None,
  }
  
  // Convert the User into JSON string.
  let serialized_user = serde_json::to_string(&user).unwarp();
  
  // print: serialized = { "username": "Max", "email": "Max@domain.com" }
  println!("serialized = {serialized_user}");

  let user = User {
    username: "Max".to_string(),
    email: "Max@domain.com".to_string(),
    phone: Some("0123454678".to_string()),
  }
  
  // Convert the User into JSON string.
  let serialized_user = serde_json::to_string(&user).unwarp();
  
  // print: serialized = { "username": "Max", "email": "Max@domain.com", "phone": "0123454678" }
  println!("serialized = {serialized_user}");
}
```

</div>

===

### Async in Rust

Rust took different approach. In Rust we have Async

===

### Compiler Error Messages

One of the great things about Rust compiler that it prints very helpful messages
(well most of the time :), and in some cases it tells you how to fix it!

===

### using moved value | Error

```rust [1-9]
fn main() {
    let name = String::from("Max");
    let max_name = name;
    print_me(name); // ERROR!
}

fn print_me(value: String) {
    println!("{value}");
}
```

<div class="fragment fade-in">

```plaintext
error[E0382]: use of moved value: `name`
 --> src/main.rs:4:14
  |
2 |     let name = String::from("Max");
  |         ---- move occurs because `name` has type `String`, which does not implement the `Copy` trait
3 |     let max_name = name;
  |                    ---- value moved here
4 |     print_me(name);
  |              ^^^^ value used here after move
```

</div>

===

### incompatible types | Error

```rust []
fn main() {
    let young = true;
    // what's `pass` type?
    let pass = if young { true } else { "false" };
}
```

<div class="fragment fade-in">

```plaintext
error[E0308]: `if` and `else` have incompatible types
 --> src/main.rs:3:41
  |
3 |     let pass = if young { true } else { "false" };
  |                           ----          ^^^^^^^ expected `bool`, found `&str`
  |                           |
  |                           expected because of this
```

</div>

===

### mismatched types | Error

```rust []
fn main() {
    let numbers = [1, 10, 33];
    print_me(numbers);
}

fn print_me(values: &[i32]) {
    println!("{values:?}");
}
```

<div class="fragment fade-in">

```plaintext
error[E0308]: mismatched types
 --> src/main.rs:3:14
  |
3 |     print_me(numbers);
  |              ^^^^^^^
  |              |
  |              expected `&[i32]`, found array `[{integer}; 3]`
  |              help: consider borrowing here: `&numbers`
```

</div>

===

### immutable variable | Error

```rust []
fn main() {
    let x = 35;
    x = 10;
}
```

<div class="fragment fade-in">

```plaintext
error[E0384]: cannot assign twice to immutable variable `x`
 --> src/main.rs:3:5
  |
2 |     let x = 35;
  |         -
  |         |
  |         first assignment to `x`
  |         help: consider making this binding mutable: `mut x`
3 |     x = 10;
  |     ^^^^^^ cannot assign twice to immutable variable
```

</div>

===

#### mutable & immutable borrow | Error

<div style="width: 140%; transform: translateX(-15%)">

```rust []
fn main() {
    // create mutable variable
    let mut name = String::from("Max");

    let name_ref1 = &name; // 1st immutable borrow
    let name_ref2 = &name; // 2nd, it's OK to have multiple immutable borrows

    let name_mut_ref = &mut name; // mutable borrow created here.. is that OK?

    println!("Hi {name_ref1}");
}
```

<div class="fragment fade-in">

```plaintext
error[E0502]: cannot borrow `name` as mutable because it is also borrowed as immutable
  --> src/main.rs:9:24
   |
5  |     let name_ref1 = &name; // 1st immutable reference
   |                     ----- immutable borrow occurs here
...
9  |     let name_mut_ref = &mut name; // ERROR
   |                        ^^^^^^^^^ mutable borrow occurs here
10 | 
11 |     println!("Hi {name_ref1}");
   |                  ----------- immutable borrow later used here
```

</div>
</div>

===

###  variable | Error


<div class="r-stack" style="font-size: 0.9em; text-align:left; width: 140%; transform: translateX(-15%)">
  <div class="fragment">

```rust [11-12,18|1-3|5-9|11-18]
fn get_user_state(user_id: u32) -> State {
    todo!()
}

enum State {
    Active,
    Disabled,
    Banned { reason: String },
}

fn main() {
    let user_state = get_user_state(10);

    match user_state {
        State::Active => println!("Hey, your account is active"),
        State::Banned { reason } => println!("your account is banned because: {reason}"),
    }
}
```

  </div>

  <div class="fragment fade-in-out">

```plaintext
error[E0004]: non-exhaustive patterns: `Disabled` not covered
  --> src/main.rs:14:11
   |
5  | / enum State {
6  | |     Active,
7  | |     Disabled,
   | |     -------- not covered
8  | |     Banned { reason: String },
9  | | }
   | |_- `State` defined here
...
14 |       match user_state {
   |             ^^^^^^^^^^ pattern `Disabled` not covered
   |
   = help: ensure that all possible cases are being handled, possibly by adding wildcards or more match arms
   = note: the matched value is of type `State`
```

  </div>
</div>

===

---

## Getting started (Rustup)

rustup is all you need to get started with Rust. It will download and install
Rust compiler and `cargo` CLI

- Linux/macOS:
```shell
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

- Windows, there's [rustup-init.exe](https://win.rustup.rs/x86_64)

===

# Cargo

Cargo is the Rust package manager. It is a tool that allows Rust packages to declare their various
dependencies and **ensure that you’ll always get a repeatable build**.

===

## `cargo` CLI

Provides you the following capabilities:

- **Create** a new package
- **Build** the package
- **Run** the package
- **Test** the package and the documentation
- **benchmarks** the package
- **generate docs** for the package
- And many more...

`cargo` CLI also support plugin system to extend it's functionality, so we can extend it to fit every need

===

## Rust Structure Overview

The smallest Rust project structure, look like this:
```shell
$ tree .
.
├── Cargo.toml    # project file (including dependencies)
└── src
    └── main.rs   # the entry point
```

<!-- ### How to add 3rd party libraries -->

---

# [crates.io](https://crates.io/)

crates.io is the official registry where community push their crates(libraries)

===

<!-- .slide: data-background-image="./crates.io.png" -->

---

## Documentations

Rust has very well-made documentation, here is a short list of the important ones:

- [The Book](https://doc.rust-lang.org/book/)
- [Rust by Example](https://doc.rust-lang.org/rust-by-example/)
- [The Standard library Docs](https://doc.rust-lang.org/std/)
- [docs.rs](https://docs.rs/) where you can find docs for libraries made by community

---

## Rust cons??

<div style="font-size: 0.9em; text-align: left">

- Learning curve  
- Not rich ecosystem yet! (in some domains)

<div style="font-size: 1.0em; font-weight: bold; text-align: center">
How to overcome these cons?
</div>

<div style="font-size: 0.9em; text-align: left">

- Rust has official channels to help you, [Discord](https://discord.gg/rust-lang)
  server, [User forum](https://users.rust-lang.org/) and
  [Internals forum](https://internals.rust-lang.org/) and the community is welcoming and willing to
  help.
- Getting involve with community to build the ecosystem if possible, or try to use Rust in area that
  have good support for such a domain, such as Web development, Embedded Development and CLI 
- Developments ..etc.

<div>

---

## Rust Foundation

Since 2021, Rust foundation was created, and it has support from big tech companies:

<div style="font-size: 0.9em;">

- **Founding Platinum**: AWS, Google, Huawei, Microsoft, Mozilla
- **Platinum**: Meta
- **Silver**: ARM, Dropbox, Toyota, 1Password, ..and many others ([source](https://foundation.rust-lang.org/members/))

</div>

The Rust Foundation is an independent non-profit organization to steward the Rust programming
language and ecosystem.

---

# Rust in numbers

===

## Who use Rust?

Rust has been adopted widely in production in many companies:

- **Canonical**: Everything from server monitoring to middleware!
- **Amazon**: Built Firecracker with Rust, it is an open source virtualization technology that
  powers AWS Lambda and other serverless offerings (
  read [more](https://aws.amazon.com/ar/blogs/opensource/why-aws-loves-rust-and-how-wed-like-to-help/))
- **npm**: Replacing C and rewriting performance-critical bottlenecks in the registry service
  architecture

===

- **Cloudflare**: We are using Rust as a replacement for memory-unsafe languages (particularly C)
  and are using it in our core edge logic.
- **Dropbox**: Optimizing cloud file-storage.
- **Figma**: Our real-time multiplayer syncing server (used to edit all Figma documents) is written
  in Rust.
- **Atlassian**: We use Rust in a service for analyzing petabytes of source code.
- **Tessel**: Single-board computing platform supports Rust applications.
- And many more... (source [1](https://www.rust-lang.org/production/users)
  , [2](https://github.com/omarabid/rust-companies))

===

## Most loved language

**Stackoverflow**:
For the sixth-year, Rust is the most loved language ([source](https://insights.stackoverflow.com/survey/2021#section-most-loved-dreaded-and-wanted-programming-scripting-and-markup-languages))

===

## Benchmarks

Rust competes with major language in terms of speed and memory consumption, not only that, but it
also ensures your code is free of bugs!

===

[nbody](https://programming-language-benchmarks.vercel.app/problem/nbody) (Input: 500000)

| lang | time  | peek-mem | compiler/runtime |
|------|-------|----------|------------------|
| cpp  | 21ms  | 1.0MB    | g++ 11.2.0       |
| rust | 25ms  | 0.7MB    | rustc 1.58.1     |
| c    | 35ms  | 1.2MB    | gcc 11.2.0       |
| go   | 54ms  | 2.8MB    | go 1.17.8        |
| java | 145ms | 39.3MB   | openjdk 17.0.2   |

===

[k-nucleotide](https://programming-language-benchmarks.vercel.app/problem/knucleotide) (input: 250000)

<div style="font-size: 0.9em">

| lang    | time   | peek-mem | compiler/runtime          |
|---------|--------|----------|---------------------------|
| c       | 35ms   | 13.0 MB  | gcc 11.2.0                |
| rust -m | 51ms   | 17.2MB   | rustc 1.59.0              |
| rust    | 81ms   | 11.6MB   | rustc 1.58.1              |
| go   -m | 224ms  | 25.2MB   | go 1.17.8                 |
| java -m | 440ms  | 83.3MB   | openjdk 19                |

</div>

===

Here is more detailed benchmarks if you're interested:
- Rust vs C ([benchmarks](https://programming-language-benchmarks.vercel.app/rust-vs-c))
- Rust vs C++ ([benchmarks](https://programming-language-benchmarks.vercel.app/rust-vs-cpp))
- Rust vs Go ([benchmarks](https://programming-language-benchmarks.vercel.app/rust-vs-go))
- Rust vs Java ([benchmarks](https://programming-language-benchmarks.vercel.app/rust-vs-java))

---

## So, why Rust is interesting?

![Rust](https://cdn.fedoramagazine.org/wp-content/uploads/2016/09/Screenshot-from-2016-09-12-08-29-02.png)

My-Note:
- comparing Rust with other languages

===

Rust is the only language that combines these things together, and it's doing it well
- Safety
- Performance
- Productivity

===


Rust also is/has:
<div style="font-size: 0.9em; text-align: left">

- Open Source
- Strongly typed language
- Active community
- Good ecosystem
- Automatically mange memory without garbage collector (GC)!
- No Seg Faults
- Zero cost abstraction
- Friendly error messages
- Smart compiler
- Support cross-completion (GNU/Linux, macOS, Windows, Android, iOS, WebAssembly, ...etc)

</div>

===

---


---

# Misc
## Testing
## Build docs with ease
## Threads in Rust
## File (I/O)
## async in Rust
## unsafe ?


