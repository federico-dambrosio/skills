# Rust Project Tools

Use this reference after inspecting the target repository. It is based on the tools present in the `grafatui` project and should be adapted to the local project rather than applied blindly.

## Discovery

Start with:

```bash
rg --files -g 'Cargo.toml' -g 'Cargo.lock' -g 'rust-toolchain*' -g 'rustfmt.toml' -g 'clippy.toml' -g 'deny.toml' -g 'taplo.toml' -g 'Justfile' -g 'justfile' -g '.github/**' -g 'README*' -g 'CONTRIBUTING*' -g 'DEVELOPMENT*'
cargo metadata --no-deps --format-version 1
```

Check `Cargo.toml` for:

- `edition` and `rust-version` (MSRV).
- Runtime and framework choices such as `tokio`, `ratatui`, `crossterm`, `clap`, `serde`, `reqwest`, and `anyhow`.
- Release/package metadata such as `package.metadata.deb` and `package.metadata.generate-rpm`.

## Core Cargo Tools

Use Cargo as the default interface:

```bash
cargo fmt --all
cargo fmt --all --check
cargo clippy --all-targets --all-features -- -D warnings
cargo test --all-features
cargo test test_name -- --nocapture
cargo build
cargo build --release
cargo run -- <args>
cargo doc --no-deps
cargo publish --dry-run
```

If a repository's CI uses simpler commands (`cargo build`, `cargo test`), still run stronger local checks when the change has meaningful risk and time allows.

## Toolchain and CI

Respect the project's MSRV. For `grafatui`, `Cargo.toml` declares Rust `1.85` and edition `2024`; GitHub Actions installs stable via `dtolnay/rust-toolchain`.

Common CI/release tools found in this project:

- `dtolnay/rust-toolchain`: install stable Rust and release targets in GitHub Actions.
- `cargo build --verbose` and `cargo test --verbose`: baseline CI quality gate.
- `release-plz`: create release PRs, changelog updates, and GitHub releases.
- `git-cliff`: generate conventional-commit changelogs.
- Conventional Commits: use `feat:`, `fix:`, `docs:`, `perf:`, `refactor:`, `style:`, `test:`, `chore:`, and `ci:`.

## Packaging and Publishing

For crates.io:

```bash
cargo metadata --no-deps --format-version 1
cargo build --release
cargo test --all
cargo publish --dry-run
cargo publish
```

For Linux packages when configured:

```bash
cargo install cargo-deb
cargo install cargo-generate-rpm
cargo build --release
cargo deb
cargo generate-rpm
```

For cross-platform release assets, check workflow target triples before changing release logic. The `grafatui` project builds:

- `x86_64-unknown-linux-gnu`
- `aarch64-unknown-linux-gnu`
- `x86_64-apple-darwin`
- `aarch64-apple-darwin`
- `x86_64-pc-windows-msvc`

## Dependency Guidance

Prefer the project's existing crates before adding dependencies:

- `anyhow`: application-level errors with context.
- `clap` and `clap_complete`: CLI parsing and shell completions.
- `clap_mangen`: man page generation.
- `tokio` and `futures`: async runtime and async utilities.
- `reqwest` with `rustls-tls`: HTTP clients without native TLS dependency.
- `serde`, `serde_json`, and `toml`: structured data.
- `ratatui` and `crossterm`: terminal UI rendering and input.
- `chrono` and `humantime`: time and duration handling.
- `directories`: platform config directories.

Add a dependency only when the standard library or existing dependencies would make the code clearly worse. Keep feature flags minimal.
