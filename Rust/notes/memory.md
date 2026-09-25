# memory.md — Rust learning context

Handoff notes for any agent picking up this repo. Read this first, then the chapter notes.

## What this repo is

`/home/jsb/cs-notebook` is a **learning notebook**, not a single project. `Rust/` holds study material
while following *The Rust Programming Language* ("the book", https://doc.rust-lang.org/book/).

```
Rust/
├── main.rs                    # scratch file, not part of any crate
├── hello_cargo/               # scratch crate (ch 1)
│   ├── Cargo.toml
│   ├── commands.md            # cargo commands (source of truth for ch 1 notes)
│   └── src/main.rs
├── guessing_game/             # scratch crate (ch 2)
│   ├── Cargo.toml             # edition 2024, rand = "0.8.5"
│   └── src/main.rs
├── notes/                     # ← the durable artifact; keep this current
│   ├── 1 Getting Started/
│   │   └── commands.md
│   └── 2 Programming a guessing game/
│       ├── notes.md           # full chapter notes (foundations)
│       └── guessing_game.rs   # reference: corrected book version
└── target/
```

## Conventions in this repo

- Chapter folders are named `N <Book Chapter Title>` under `notes/`.
- Each chapter folder holds a `notes.md` for prose + a `.rs` reference file for working code.
- The `hello_cargo/` and `guessing_game/` directories are **throwaway scratch crates** for `cargo run`.
  `notes/` is the thing worth preserving.
- Notes style the user likes: concept → small code snippet → table of rules → "gotchas".
  Markdown tables and short annotated snippets land better than long prose.

## Progress log

- **Ch 1 — Getting Started:** done. Notes in `notes/1 Getting Started/commands.md`
  (cargo commands, `--release`, `cargo new`, project layout).
- **Ch 2 — Programming a Guessing Game:** done. Notes in
  `notes/2 Programming a guessing game/notes.md`. Covers immutability, `mut`, shadowing, prelude
  and `use`, `String::new()`, `stdin().read_line(&mut ...)`, `&` vs `&mut`, `Result`/`Ok`/`Err`,
  `.expect()` + `must_use` warning, `{}` placeholders, crates/packages/`Cargo.toml`, `rand`,
  `thread_rng`, `Ordering`, `match` arms + exhaustiveness, `parse` + trim, static typing.
- **Ch 3 — Common Programming Concepts** (variables, data types, functions, control flow,
  ownership preview): **not started**.

## Environment facts

- Linux, zsh, workspace root `/home/jsb/cs-notebook`.
- `rustc`/`cargo` available; `cargo build` runs in `Rust/guessing_game`.
- `guessing_game/Cargo.toml`: `edition = "2024"`, `rand = "0.8.5"` (pinned — see gotcha below).

## Gotchas / things that bite

1. **rand version split.** With rand `0.8.x`, `rand::thread_rng().gen_range(1..=100)` is correct.
   rand `0.9+` renamed these to `rand::rng()` / `random_range()`. If a build fails on missing
   `thread_rng`, check `Cargo.toml` before rewriting the code.
2. **`gen_range` needs the trait.** `use rand::Rng;` must be present or the method isn't found —
   a trait method without the trait in scope is an error, not a warning.
3. **`std::io` and `std::cmp::Ordering` are NOT in the prelude.** They always need a `use` (or a
   fully qualified path).
4. **`new` is not a keyword.** `Type::new()` is convention; `::` = associated function, `.` = method.
5. **Folder names contain spaces** (`notes/1 Getting Started/`). Works fine for Markdown, but quote
   paths in shell commands and never `cargo new` inside them unquoted.
6. **`read_line` keeps the `\n`** — forget `.trim()` before `.parse()` and parsing fails at runtime.
7. **Bare `Result` = warning, not error** (`unused_must_use`). `expect`/`unwrap` panic on `Err`;
   prefer `match`/`?` once the user reaches ch 9.

## Known gaps in the user's current code

`Rust/guessing_game/src/main.rs` as of this writing has: `use rand:Rng` (single colon — syntax
error) with no semicolon, both `use`s placed inside `fn main` instead of at the top, and no `loop`
so the game reads one guess and exits before "You win!" is reachable. These are listed in
`notes.md` under "Bugs spotted". Offer to fix rather than silently rewriting — the user is mid-lesson.

## How the user learns (pattern analysis)

Observed from their scribbled brief:

- **Annotates working code incrementally.** Learns by building a small program and interrogating
  each line ("how we receive user input", "address we give where to store it").
- **Asks mechanism-first questions** — *how* the code does a thing, before *why* it's designed that
  way. Answer the mechanics concretely, then give the rationale.
- **Compresses concepts into one-line mnemonics** ("Result value is returned and enum and each state
  a variant"). These are cues to themselves, not finished notes — expand them into full sentences
  with proper terminology.
- **Occasional terminology slips** worth correcting gently but explicitly: calling `new` a keyword,
  calling `Ordering` a crate (it's std), writing `rand:Rng`. Flag these as they are
  high-value corrections at this stage.
- **Front-loads breadth over depth** — mentions many topics in one pass ("power rust features
  discuss later"). Maintain a "What to learn next" list; don't fully teach deferred topics when
  the user explicitly defers them.
- **Wants a visible artifact.** They ask for notes so they become "my rust foundation" — write for
  future re-reading, with tables and snippet-first structure.
- **Prefers the book's flow.** Notes should align with the chapter the user is on and use the
  book's example code, plus version caveats where the book lags reality.

## Advice for the next agent

- When the user dumps shorthand, **expand it into sections** mapped back to their own words
  (the "your scribbles → the concepts" table in `notes.md` is the pattern to reuse).
- Keep `notes/` in sync whenever a scratch crate changes.
- Ask before rewriting lesson code; the wrong-but-instructive version still has value.
- When a chapter is finished, update the Progress log here.
