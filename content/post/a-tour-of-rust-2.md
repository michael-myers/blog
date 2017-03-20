+++
Description = "A Tour of Rust, Pt. 2"
Categories = ["rust", "programming"]
date = "2017-03-20T22:56:03-05:00"
title = "A tour of Rust, Pt. 2"
Tags = ["rust", "programming", "exercises"]
+++

In my last post, I talked about what I think is the significance of the programming language Rust, and why I wanted to try learning it. Today, I take a look at [Exercism.io](http://exercism.io) (a sort-of social network for programming exercises) and its series of [Rust challenges](http://exercism.io/languages/rust/about). These are definitely easy problems (so far); the focus is on learning how to solve them in a language that is new to you. We'll see where I tripped up while trying to grasp some of the Rust language features, and some Rust I've learned so far.

## Exercism.io Setup and Workflow

Assuming you also use MacOS with Homebrew, it's just a couple of steps:

```sh
$ brew update && brew install exercism
$ exercism configure --key=YOUR_API_KEY
$ exercism configure --dir=~/Documents/Exercism
```
And then each new programming exercise is fetched this way:

```sh
$ exercism fetch rust		# will download the next exercise, "whatever"
$ cd ~/Documents/Exercism/whatever
$ mkdir src && touch src/lib.rs   # then, work on your solution in lib.rs
```

Each exercise asks you to write a Rust library that exports a function or two, such that you implement some behavior instructed in the README.md for that exercise. The folder structure provided is that of a [Rust "crate"](http://doc.rust-lang.org/stable/book/crates-and-modules.html), which is what Rust (or rather, its build tool, Cargo) calls a source package. You will define a `pub fn whatev()` in `src/lib.rs`, which is the filename convention for a Rust *library* crate (as opposed to an *executable* crate, which would have a `fn main()` defined in a `src/main.rs`). `Cargo.toml` is the manifest file that defines the crate: the version string, the dependencies, author.

Each challenge comes with unit tests in `tests/whatever.rs` (technically, integration tests. Rust allows you to write the unit tests inline with the source files themselves). [You can run the tests using Rust's build tool, Cargo](https://doc.rust-lang.org/book/testing.html):

```sh
$ cargo test	# both compiles your library and runs the tests
```

If you *just* wanted to compile, you could run `cargo build`. Were this an executable crate rather than a library crate, you could also `cargo run`, but with a library crate `run` has no meaning, so we `cargo test`. Note: if you had noticed that the binaries are rather large, that is because Cargo builds debug binaries by default. For release, you would use `cargo build —release`.

Once all of the unit tests pass for an exercise, you can submit your solution like so:

```sh
$ exercism submit src/lib.rs
```

## Rust Debugging in the VS Code editor

Although I discussed setting up a Rust development environment in the last post, Andrew Hobden has also documented [the setup and use of other tools in the Rust development toolchain](https://hoverbear.org/2017/03/03/setting-up-a-rust-devenv/), and his writeup may be worth a look. For me right now, I don't want the added headache of trying to work with any alpha or "nightly build" features of Rust. What was important for me was [debugging in VS Code](https://code.visualstudio.com/Docs/editor/debugging), so I appreciated his help with that. While I previously had no luck getting the "Native Debug" VS Code extension to work, using Andrew's instructions, I did get the "LLDB Debugger" extension to work.

*But do you need to manually recreate .vscode/launch.json and .vscode/tasks.json for every project you want to debug? That blows* – Well sort of. In VS Code you can click the debug icon in the sidebar and then the settings wheel icon in the debug pane that appears, and VS Code will create a mostly-complete launch.json file to which you just have to add:

```
"preLaunchTask": "cargo",
"sourceLanguages": ["rust"]
```

And of course, you'll have to fix the final part of the path for "program" (your debug target). So there isn't that much to do manually for each new project. But when you tell VS Code to run a "preLaunchTask", as above, you then have to define that "task" in a `.vscode/tasks.json` file, but it's the same every time, just copy and paste it from your last project. A hassle compared to debugging with a real IDE, but a minor hassle at least.

## Exercism Exercises 1-10

### 1: Hello World, and strings in Rust

It looks like the developers of this challenge changed their answer format somewhere along the way during development, and now it actually contains conflicting instructions. Fortunately, this is the only challenge with this problem, but ignore the README.md this time as well as the GETTING_STARTED.md. As with most of these challenges, the most important file is `tests/hello-world.rs` which defines the Cargo unit tests and gives the guiding examples of what your code is supposed to produce. In this case, it is very simple, you just need to produce the string "Hello, World!" using a function called `fn hello`.

But what is the correct function declaration for `fn hello`? First, it has to be a function that is *published* by your Rust library for external callers, thus it is `pub fn hello`.

It is not taking any arguments (despite what the muddled insructions state), so it is `pub fn hello()`.

And it returns a string, so it is…uh-oh. Here, Rust makes this harder than you might expect. There is the primitive type for representing strings, `str`, and then there is a `String` type (from the Rust standard library). They seem similar and completely redundant at first, but their purposes and usages are different in a variety of ways that will trip you up if you don't understand what each one means. This duality of Strings and string literals is essential to understand, and it is poorly explained (if explained at all) in every Rust tutorial I've seen. If people need to write [long posts explaining the difference](http://hermanradtke.com/2015/05/03/string-vs-str-in-rust-functions.html), I think the language documentation could be doing a better job here.

Complicating matters, `str` is used synonymously in Rust documentation with "string" and "string literal", and a reference to a subset of a `String` is an `&str`, a.k.a. "string slice". In fact, a function that takes a `&str` can be passed a `&String` (a concept in Rust called *coercion*, i.e. implicit type conversion), but a function that takes a `&String` cannot be passed a `&str`. Wow. Confused yet? Just wait until you try to concatenate two strings. We'll get to that later.

If you choose to use `String`, the return type is simple to understand, but you have to build a `String` instance out of a string literal using `.to_string()` or `String::from`, which is non-obvious:

```rust
pub fn hello() -> String {
  "Hello, World!".to_string()  // alternatively, String::from("Hello, World!")
}
```

If you choose instead to use `str`, the actual returned value needs nothing special, but the return type is by [*borrowed reference*](http://intorust.com/tutorial/shared-borrows/) (hence the ampersand) and requires a *lifetime* specifier, something unique to Rust:

```rust
pub fn hello() -> &'static str {
    "Hello, World!"
}
```

This is to say, `hello()` returns a reference to an immutable string literal. In other words, a pointer to the string "Hello, World!" and the caller of hello() cannot change that string using a dereference of this pointer. The reference is valid for `static`, a lifetime duration defined as the "duration of the entire program." This is basically a guarantee to the caller that this reference will always be valid. String literals will always get a static lifetime because they're hard-coded in the compiled Rust binary's data section; they are never deallocated.

So, with a simple HelloWorld example we've had to introduce ourselves to the three big concepts unique to Rust: **ownership**, **reference borrowing**, and **lifetimes**. We've also tripped over the `str`/`String` duality and the concept of [coercion](https://github.com/rust-lang/rfcs/blob/master/text/0401-coercions.md). As we struggle to comprehend these concepts, they'll be responsible for the majority of our compile-time errors. This is the Rust learning curve.

### 2: Gigasecond, and including external crates

Hint, for this one, you'll be needing the [Chrono](https://github.com/chronotope/chrono) crate, because the Rust standard library currently has no library for handling the concept of time. Your lib.rs file begins with:

```rust
extern crate chrono;
use chrono::*;
```

And your Cargo.toml file declares this dependency as well:

```toml
[dependencies]
chrono = "0.2"
```

When you `cargo test`, Cargo will automatically fetch and install the crate "Chrono" for you. Nice! Now you can add seconds to times and compare times to one another.

The instructions for this challenge may mislead you to try to use the caret as an exponentiation operator:

> A gigasecond is 10^9 (1,000,000,000) seconds.

Yes, but Rust (like C before it) lacks an exponentiation operator. Not only does `10^9` *not* define the value `1_000_000_000`, it also doesn't generate a compile-time error. Instead, it is interpreted as the bitwise XOR of 10 and 9: in other words, 10^9 equals 3 (surprise, LOL). Again, the official Rust documentation ("The Rust Programming Language") is a bit lacking with its complete absence of explanation of the operators that Rust actually has and does not have, a fundamental part of any language. Instead, you should consult the "Rust Language Reference" for this information. That said, if you really want to do exponentiation, several of the primitive types have exponentiation methods: the floating point types `f32` and `f64` which offer the `powi(i32)` method, and the integer types `i32` and `i64` which offer `pow(u32)`.

```rust
let ten = 10_i64;
ten.pow(9)  // this is 1,000,000,000
```

### 3: Leap, and the modulus operator

There is little to learn from this exercise except the proper use of the % operator, which, again, was up to you to find in the Language Reference. It's an "either you know this trick or you don't" challenge, but popular in whiteboard programming questions in job interviews, and occasionally useful in real life. Example snippet:

```rust
// On every year that is evenly divisible by 4:
    if candidate_year % 4 == 0 {
        // Except every year that is evenly divisible by 100:
        if candidate_year % 100 == 0 {
```

### 4: Raindrops, and the modulus operator again

This is a simple integer-to-string (hint: `some_value.to_string()`) and integer-factoring challenge. Again, the modulo operator is all you need, and this exercise fails to add any new lesson really.

### 5: Bob, and iterators

Given some `&str prompt` you can iterate over every character in the string in a for loop, without having to use pointers or indices as a C programmer is tempted to do:

```rust
for character in prompt.chars() {
	// do stuff to each character
}
```

And in fact, it is basically impossible to loop across a `str` in any other way, because you cannot use array indexing on a `str` as you might with a string in C, and create a for loop that ranges from prompt[0] to prompt[prompt.len()]. Even for Rust types where that pattern *is* possible, it is discouraged: find your loop ranges using iterators, which are returned by methods like `.chars()` or `.iter()`. The code above automatically turns `character` into a value of type `char` because `prompt.chars` is a range of `char` values.

Rust's `char` type has some handy methods, for example: `if character.is_alphabetic()` and `if character.is_uppercase()`. 

### 6: Beer Song, string concatenation, and the match statement

String concatenation in Rust is completely bonkers:

```rust
let a = "foo";
let b = "bar";

println!("{}", a + b); 							// invalid
let c = a + &b;  								// invalid
let c: String = a + b.to_string();				// invalid
let c: String = a.to_string() + b.to_string(); 	// invalid

let c: String = a.to_string() + b;  			// valid!
let c: String = a.to_string() + &b.to_string();	// valid!
c.push_str(" more stuff on the end");			// valid!
```

The strings `a` and `b` here are `str` instances. The `str` type lacks any kind of concatenation operator, so you can't use a `+` to concatenate them *when the left operand is a `str`*. However *when the left operand is a `String`* you **totally can** use the `+` because `String` **does** have the concatenation operator. 

The `String` type is growable, whereas `str` is an annoyingly restricted type of data that mucks up everything it touches. You can't build a `str` out of other `str`; you can't even build a `String` out of two `str` without bending yourself into a pretzel. You may seek to avoid `str` altogether, but you can't. Because every Rust string literal *is* a `str`, we are forced to work with both `str` and `String`, upconverting the former to the latter with `.to_string()`, and/or connecting them onto the end of a `String` with its `.push_str()` method.

But at least you can use the `match` keyword to help with this challenge:

```rust
// Form the appropriate English words to refer to the bottle or bottles:
fn bottles(quantity: u32) -> String {
    match quantity {
        0 => "No more bottles".to_string(),
        1 => "1 bottle".to_string(),
        _ => quantity.to_string() + " bottles",
    }
}
```

That's a lot cleaner than an if—else-if—else would have been.

### 7: Difference of Squares

This one should be a review, you can use iterators (in the form of for-loop ranges), and exponentiation isn't necessary if you just want to do squares:

```rust
let somevalue = 123;
let mut square;
square = somevalue * somevalue;
```

### 8: Sum of Multiples

Another review challenge. Tests you again on using iterators, and the modulus operator ("is a multiple of x" is synonymous with "is cleanly divisible by x"). You might use nested loops, or maybe something fancier like closures. I think this post is long enough without addressing that concept.

### 9: Grains, and the panic macro

Back on exercise 2 we learned how to do exponentiation in Rust, so that's half of this challenge. The other hint is that the unit tests are testing for error conditions indicated by "panic." The way to panic in Rust is via the panic macro:

```rust
if s < 1 || s > 64 {
	panic!("Square must be between 1 and 64");
}
```

### 10: Hamming, unwrap, and the Result type

And finally (for now), we learn how to multiplex a return value and an error condition into [one type, `Result`.](https://doc.rust-lang.org/std/result/) 

If we choose the return value for our function to be `-> Result<i32, &'static str>` then we are declaring that we might return either `Ok(123)` or `Err("Something went wrong")`.

Some of the unit tests are checking to see if the returned condition is an error of any kind: `.is_err()`. Using `Err()` satisfies that check.

## Rust Initial Impressions

So after getting beyond "Hello world" and trying a few exercises, my initial impressions of Rust as a language are that its strictness is its defining characteristic. You could even say it's a pain, honestly, not what I could call a joy to work in (although speaking ill of Rust invites the fans to show up and blame you for failing to love it). The payoff doesn't have to be rapid prototyping joy, though, it just has to be the more secure code that you are ostensibly creating by being so strict and explicit about everything. That's okay too.

| The Good                                 | The Bad                                  |
| ---------------------------------------- | ---------------------------------------- |
| Cargo build tool                         | The official documentation for Rust's language and standard library |
| Passionate community                     | Condescending comments like "I can tell you're an imperative language guy" when you don't use closures |
| `rustfmt` for automated code style enforcement (`cargo fmt`) | Learning curve for the errors emitted by the Rust compiler |
| Rust can be debugged with `rust-lldb` or `rust-gdb` and this mostly works within VS Code | Your errors will all be compile-time anyway, for better or worse |
| Expressive method names like .is_alphabetic() are a welcome improvement to C standard lib | Any time you have to use strings (String vs str, string concatenation, etc.) you will wonder if Rust will ever catch on |

