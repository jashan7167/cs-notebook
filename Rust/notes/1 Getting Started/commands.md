# Cargo Commands

| Command | What it does |
| --- | --- |
| `cargo run` | Compiles the code and then runs the resulting binary. |
| `cargo build` | Compiles the code only, producing a binary in `target/debug/`. |
| `cargo check` | Type-checks the code without producing a binary — much faster than `build`. |

## Release mode

Add `--release` to build with optimizations (slower compile, faster runtime). Output goes to `target/release/`.

```sh
cargo run --release
cargo build --release
cargo check --release
```

**In short:** `check` = fastest, just verifies; `build` = compile; `run` = compile + execute. Use `--release` for optimized code you want to ship or benchmark.

---

# Creating a New Project

```sh
cargo new hello_cargo   # creates a new binary project + git repo
cargo init              # initializes cargo in an existing directory
```

Project layout:

```
hello_cargo/
├── Cargo.toml      # package manifest (name, version, dependencies)
└── src/
    └── main.rs     # entry point for a binary crate
```

`Cargo.toml`:

```toml
[package]
name = "hello_cargo"
version = "0.1.0"
edition = "2021"

[dependencies]
```

Notes:
- `cargo new --lib name` creates a library crate (`src/lib.rs`).
- Code always lives in `src/`; the top-level folder is just the project root.
- `target/` holds build output and is safe to delete (regenerated on next build).
- Add `Cargo.lock` and `target/` to version control ignore — commit `Cargo.toml`.
