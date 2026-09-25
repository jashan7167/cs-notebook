# 2 — Programming a Guessing Game

Foundations from building the guessing game. These are the raw concepts behind the code, written out properly.

---

## The target program (annotated)

```rust
use rand::Rng;            // the Rng *trait* must be in scope for .gen_range() to exist
use std::cmp::Ordering;   // the Ordering enum is NOT in the prelude
use std::io;              // std::io is NOT in the prelude either

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1..=100);

    loop {
        println!("Please input your guess.");

        let mut guess = String::new();

        io::stdin()
            .read_line(&mut guess)
            .expect("Failed to read input");

        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };

        println!("You guessed: {guess}");

        match guess.cmp(&secret_number) {
            Ordering::Less => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal => {
                println!("You win!");
                break;
            }
        }
    }
}
```

Everything below explains one piece of it.

---

## 1. Variables are immutable by default

```rust
let x = 5;
x = 6;          // ERROR: cannot assign twice to immutable variable
```

Rust makes immutability the default. If a value never changes, the compiler can reason about it freely, and bugs from accidental mutation disappear. To opt in:

```rust
let mut x = 5;  // `mut` = this binding may be reassigned
x = 6;          // fine
```

Rule of thumb: **start without `mut`, add it when the compiler asks.** Fewer `mut`s = fewer surprises.

---

## 2. Shadowing

```rust
let guess = String::new();              // guess is a String
let guess: u32 = guess.trim().parse().unwrap();   // a NEW `guess`, type u32
```

The second `let` **shadows** the first. The old `String` isn't destroyed — it just becomes unreachable by that name in this scope.

Two things shadowing gives you that `mut` cannot:

| Shadowing (`let x = ...`) | Mutation (`let mut x`) |
| --- | --- |
| Can change the **type** | Type is locked at declaration |
| Creates a new binding | Reuses the same binding |
| Old value still lives until scope ends | Same value is overwritten |

This is why the guessing game can reuse the name `guess` for both the `String` input and the parsed `u32` number — a shadowed variable is a clean way to transform a value while keeping a readable name.

---

## 3. Bringing names into scope: `use` and the prelude

Every Rust program implicitly starts with the **prelude** — a short list of items from the standard library injected into scope automatically. The common ones:

- Types: `String`, `Vec`, `Box`, `Option`, `Result`, `Iterator`
- Variants: `Some`, `None`, `Ok`, `Err`
- Traits: `Clone`, `Copy`, `Default`, `Drop`, `Fn`, `PartialEq`, `PartialOrd`, `Eq`, `Ord`, `From`, `Into`, `ToString`, `Debug`, `Display`

**`std::io` and `std::cmp::Ordering` are not in the prelude**, so you have to bring them in yourself. Three ways:

```rust
// 1. Import the module, then qualify
use std::io;
io::stdin()

// 2. Import the item directly
use std::io::stdin;
stdin()

// 3. Fully qualified inline — no `use` at all
std::io::stdin()
```

Combining, the idiomatic form for a module plus a trait is:

```rust
use std::io::{self, Write};
```

`self` here means "also import `io` itself," so both `io::stdin()` and `Write`-trait methods work.

> **Placement:** `use` normally goes at the top of the file. Inside `fn main()` it also works, but it is scoped to that function — unusual style, and easy to forget.

---

## 4. `String::new()` — `::` is not a `new` keyword

`new` is **not** a keyword in Rust. It is just a naming **convention**: types commonly provide an associated function called `new` that returns a default/empty value.

- `::` reaches into a type's namespace — an **associated function** (no `self`, so not a method).
- `.` calls a **method** on a value (has `self`).

```rust
let s = String::new();   // :: → associated function, no instance needed
s.len();                 // .  → method, needs the instance
```

`String` is a standard-library type: a **growable, heap-allocated, UTF-8 encoded** text buffer. Contrast with `&str`, which is a borrowed, fixed-length string slice (a view into UTF-8 bytes). `String::new()` gives you an empty string ready to be filled.

---

## 5. How we receive user input

The chain `io::stdin().read_line(&mut guess)` is three things:

| Step | Meaning |
| --- | --- |
| `io::stdin()` | Returns a `Stdin` **handle** to the process's standard input stream |
| `.read_line(...)` | A method on that handle that blocks until the user presses Enter |
| `&mut guess` | The **address of the buffer** where the typed bytes should be stored |

The signature:

```rust
pub fn read_line(&self, buf: &mut String) -> io::Result<usize>
```

So `read_line`:
- takes a **mutable reference to a `String`** (an existing buffer to append into),
- returns how many bytes were read, wrapped in a `Result`,
- on success appends the typed line **including the trailing `\n`** — which is why we later call `.trim()`.

We hand over `&mut guess` rather than `guess` because we want the function to fill *our* buffer. Passing `guess` by value would move ownership into the function and we could not use it afterwards.

---

## 6. References: `&` and `&mut`

A **reference** lets code use a value without taking ownership. References, like bindings, are **immutable by default**.

```rust
fn length(s: &String) -> usize { }          // shared / immutable borrow
fn add_text(s: &mut String) { }             // exclusive / mutable borrow
```

- `&T` — you may read, not write. Many readers allowed at once.
- `&mut T` — you may read and write. Only one at a time, and no readers while it exists.

Rust's borrow checker enforces this at compile time, which is why references feel "easy to use": data races and use-after-free become compile errors instead of runtime mysteries.

`&mut guess` in `read_line` is the mutable reference that lets the function append to `guess`.

---

## 7. `Result` — errors are values

Rust has no exceptions. Fallible operations return a `Result`, which is an **enum**:

```rust
enum Result<T, E> {
    Ok(T),    // variant: success, carries the value
    Err(E),   // variant: failure, carries the error
}
```

`io::Result<usize>` is a type alias for `Result<usize, io::Error>`. Variants are how an enum represents "one of several states"; here, exactly two: success or failure.

To get the value out you must handle both variants. Options:

```rust
.expect("Failed to read input")   // Ok → value; Err → panic with this message
.unwrap()                         // Ok → value; Err → panic with a generic message
match r { Ok(v) => v, Err(e) => ... }  // handle it properly
?                                 // inside a function returning Result: propagate
```

> **Warning without handling:** `Result` is marked `#[must_use]`. Ignoring the return value compiles but emits
> `warning: unused Result that must be used`. `expect`/`unwrap` silence it — at the cost of panicking instead of recovering.

---

## 8. Placeholders: `{}`

`println!` is a macro (variadic + compile-time checked format string). Placeholder syntax:

| Placeholder | Prints |
| --- | --- |
| `{}` | `Display` — the human-readable form |
| `{:?}` | `Debug` — the programmer-facing form (used for structs/enums) |
| `{:#?}` | pretty-printed `Debug` |
| `{name}` | named/inline variable capture (Rust 2021+): `println!("{guess}")` |
| `{:>5}` / `{:05}` | width / zero-padding |

The number of `{}` must match the number of arguments, and the types must implement the matching trait — both verified at **compile time**.

---

## 9. Crates, packages, and `Cargo.toml`

- **Crate** — a compilation unit. Either a *binary crate* (has `fn main`, produces an executable) or a *library crate* (reusable code, e.g. `src/lib.rs`).
- **Package** — a bundle of one or more crates plus a `Cargo.toml`. Cargo builds packages.
- **`Cargo.toml`** — the manifest declaring the package and its dependencies.

```toml
[package]
name = "guessing_game"
version = "0.1.0"
edition = "2024"

[dependencies]
rand = "0.8.5"        # an external crate from crates.io
```

Adding a dependency makes its crates available to `use` in your code, and Cargo fetches/compiles them automatically on the next `cargo build`.

---

## 10. Generating a random number

```rust
let secret_number = rand::thread_rng().gen_range(1..=100);
```

| Piece | What it does |
| --- | --- |
| `rand` | The external crate providing random-number generation |
| `thread_rng()` | Returns a `ThreadRng` — a handle to a **thread-local** random number generator, seeded from the OS. Fast, not `Send`, so it stays on the current thread of execution. |
| `gen_range(1..=100)` | Produces a value in the range `1..=100` (inclusive both ends). `1..100` would be exclusive of 100. |
| `use rand::Rng;` | `gen_range` is a **trait method** of `Rng`. Without the trait in scope, the method does not exist as far as the compiler is concerned. |

> **Version gotcha:** your `Cargo.toml` pins `rand = "0.8.5"`, where `thread_rng()` / `gen_range()` are correct. In `rand 0.9` these were renamed to `rand::rng()` / `random_range()`, and `gen_range` is deprecated. Keep the pinned version or the book's code and the docs will disagree.

---

## 11. `Ordering` and the `match` expression

`cmp` compares two values and returns `Ordering` (an enum from `std::cmp`):

```rust
enum Ordering {
    Less,
    Greater,
    Equal,
}
```

`match` is an **expression** — it produces a value, so it can be assigned to a `let` or returned. Its structure:

```rust
match value {
    Pattern1 => expression1,   // an "arm"
    Pattern2 => expression2,
    _        => expression3,   // wildcard: matches anything
}
```

Rules:
- Each arm is `pattern => expression`, arms separated by commas.
- The patterns must be **exhaustive** — every possible value must be covered, or it's a compile error. (This is why matching on `Ordering` without `Equal` fails.)
- Every arm must evaluate to the **same type**, since the whole `match` yields one value.
- `_` is the catch-all; use it when you deliberately don't care about the rest.

This exhaustiveness checking is one of Rust's headline features — add a variant to an enum later and the compiler will point at every `match` that now needs updating.

---

## 12. `parse()` and the static type system

Rust's type system is **static** (all types resolved at compile time), **strong** (no implicit conversions), and **inferred** (you rarely write types). Inference needs a clue for `parse`, because the same text could become many types:

```rust
let guess: u32 = guess.trim().parse().expect("Please type a number!");
//         ^^^^ tells the compiler which type to parse into
```

- `.trim()` removes leading/trailing whitespace — necessary because `read_line` keeps the `\n`.
- `.parse()` converts a `&str` into another type, returning `Result<T, T::Err>`. It only succeeds if **every character can be converted** to the target numeric type; anything else is an `Err`, not a silent `0`.
- The turbofish `"42".parse::<u32>()` is the alternative way to state the target type.

Default numeric types when nothing constrains them: `i32` for integers, `f64` for floats.

---

## Mapping: your scribbles → the concepts

| Your note | Concept | Section |
| --- | --- | --- |
| "variables are immutable in rust" | Immutable by default, opt in with `mut` | 1, 2 |
| "String new is a string type by std lib growable utf-8 encoded bit of text" | `String` = growable, heap, UTF-8 | 4 |
| "new keyword in rust" | ⚠️ `new` is a *convention*, not a keyword; `::` = associated function | 4 |
| "how we receive user input / address we give where to store the user input" | `stdin()` handle + `read_line(&mut String)` | 5 |
| "it is a reference and references are ... immutable and need to have &mut to make it mutable" | `&T` vs `&mut T` borrows | 6 |
| "a Result value is returned and enum and each state a variant ok and err" | `enum Result<T,E> { Ok(T), Err(E) }` | 7 |
| "err is catched by expect without expect gives warning in compile" | `.expect()` panics; bare `Result` → `unused_must_use` warning | 7 |
| "placeholders are {}" | `{}` = `Display`, `{:?}` = `Debug` | 8 |
| "add rand crate what are crates" | Crate / package / `Cargo.toml` | 9, 10 |
| "thread_rng giving us the current thread of execution" | thread-local RNG handle | 10 |
| "adding the ordering crate" | `std::cmp::Ordering` — a std enum, not a crate | 11 |
| "a match expression it is made up of arms" | `match` arms, exhaustiveness | 11 |
| "power rust features discuss later" | Exhaustive enums, borrow checker, no-GC safety | 11 |
| "strict static type system in rust" | Static + strong + inferred | 12 |
| "Shadowing is also introduced" | `let` re-binding, can change type | 2 |
| "parse works on characters that can be convt to numbers otherwise no" | `parse` returns `Result`, never silently fails | 12 |

---

## Bugs spotted in the current `main.rs`

These are worth fixing — the file as written does not compile:

1. **`use rand:Rng`** — single colon is a syntax error. Needs `use rand::Rng;` and belongs at the **top of the file**, not inside `main`.
2. **Missing semicolon** after that `use`.
3. **`use std::cmp::Ordering;` inside `fn main()`** — legal but inconsistent; move it to the top with the others.
4. **No `loop`** — the program reads one guess and exits, so it never reaches "You win!". A `loop` (with `break` on `Equal`) is what the game needs.
5. **`<` `>`** — `std::io::stdin()` is fine fully qualified, but `Ordering` is used unqualified, so the `use` is required.
6. **Style:** `match guess.cmp(&secret_number){` — rustfmt wants a space before `{`.

---

## What to learn next

- `loop` / `while` / `for`, `break`, `continue`
- `enum` + `match` in depth: `Option<T>`, `if let`, `while let`
- Structs, methods (`impl`), and `#[derive(Debug)]`
- Ownership, borrowing, and lifetimes (the real payoff of the `&mut` you met here)
- Error handling properly: `?`, custom error enums, `thiserror` / `anyhow`
- Modules and the filesystem layout: `mod`, `pub`, `use crate::...`
- Tests: `#[cfg(test)]`, `cargo test`
