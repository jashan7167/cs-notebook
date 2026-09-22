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
