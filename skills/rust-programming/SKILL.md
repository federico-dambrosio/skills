---
name: rust-programming
description: Rust programming best practices for implementing, reviewing, refactoring, testing, linting, packaging, and releasing Rust crates and applications. Use when Codex works on Rust files, Cargo projects, Cargo.toml metadata, async Tokio code, CLI/TUI applications, serde models, release automation, rustfmt/clippy/test/doc quality gates, or Rust architecture and error-handling decisions.
---

# Rust Programming

## Operating Mode

First inspect the local project before choosing conventions. Read `Cargo.toml`, tool config files, README/development docs, and CI workflows. Prefer the project's existing edition, MSRV, dependency choices, module layout, error style, and test style over generic Rust preferences.

Keep changes small and idiomatic. Avoid broad rewrites, speculative abstractions, and dependency additions unless they clearly reduce complexity or match existing project direction.

Read `references/project-tools.md` when you need a command checklist or release/package tool context.

## Design Approach

Model the domain with explicit types and narrow ownership. Prefer:

- `struct` and `enum` variants that make invalid states hard to express.
- Borrowed inputs (`&str`, `&Path`, slices) when a function does not need ownership.
- Owned values at clear boundaries: config loading, parsed CLI args, async tasks, caches, and stored application state.
- Small modules that follow existing responsibility boundaries.
- Pure helper functions for parsing, mapping, formatting, and calculations so behavior is easy to test.

Use traits only when there is an actual polymorphic boundary or a test seam already present. Do not hide simple concrete code behind traits for style alone.

## Error Handling

Follow the crate's current error strategy. In application binaries, `anyhow::Result` with contextual messages is usually fine. In libraries or public APIs, prefer typed errors when callers need to branch on failure modes.

Add context at I/O, network, parsing, and external boundary failures. Avoid swallowing errors or converting them to vague strings unless the UI boundary specifically requires a display message.

Use `?` for propagation and keep recovery paths explicit. Avoid `unwrap` and `expect` outside tests unless an invariant is locally proven and the panic message documents that invariant.

## Async and Concurrency

Keep async boundaries clear. Use Tokio facilities already present in the project, avoid blocking calls inside async functions, and make cancellation-safe behavior explicit when spawning or joining tasks.

Protect shared mutable state with the smallest suitable primitive. Prefer ownership transfer or channels before `Arc<Mutex<_>>` when the data flow is simple.

## CLI, Config, and Serialization

For CLI apps, keep `clap` types as the input boundary and convert into domain/config types before deeper logic. Validate user-facing arguments close to parsing, and keep help strings accurate.

For `serde` models, type optional and defaulted fields explicitly. Use `#[serde(default)]`, `rename`, and custom deserializers instead of brittle post-processing when parsing external JSON/TOML formats.

## TUI and Rendering

Separate state mutation from rendering. Rendering functions should mostly derive widgets from immutable state; input handlers and app methods should own state transitions.

For Ratatui-style code, test layout math, state transitions, parsing, color mapping, and text formatting separately from terminal I/O. Keep terminal setup/teardown isolated and resilient to early returns.

## Documentation

Document public APIs with Rust doc comments (`///`). Include examples only when they clarify non-obvious behavior and can be kept correct. Keep private comments rare and focused on invariants or tricky algorithms.

When changing CLI flags, config files, dashboard formats, public structs, release metadata, or user-visible behavior, update README/development docs and examples that depend on it.

## Testing

Changed behavior requires tests at the closest useful level:

- Unit tests for pure parsing, mapping, calculations, state transitions, and error cases.
- `#[tokio::test]` for async behavior.
- Integration tests when behavior crosses crate/module boundaries or exercises CLI-facing workflows.
- Example fixtures when supporting external formats such as JSON dashboards or TOML config.

Test edge cases and failure paths, not only the happy path. Avoid network-dependent tests unless the project already has a deterministic local fixture or mock pattern.

## Quality Gate

Before finishing Rust code changes, run the project's quality gate. Prefer the commands already used by the repository; otherwise use:

```bash
cargo fmt --all --check
cargo clippy --all-targets --all-features -- -D warnings
cargo test --all-features
cargo build --release
```

Use `cargo fmt` without `--check` when you are allowed to modify formatting. For publishable crates, run `cargo doc --no-deps` when public API docs changed and `cargo publish --dry-run` before release packaging.
