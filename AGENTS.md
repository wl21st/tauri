# Tauri Development Guide

This file contains guidelines and commands for agents working on the Tauri codebase.

## Project Overview

Tauri is a framework for building tiny, blazingly fast binaries for all major desktop platforms using Rust + WebView. The codebase contains:

- **Rust crates**: `crates/tauri*`, `packages/cli` (Rust)
- **JS/TS packages**: `packages/api`, `packages/cli` (Node wrapper)
- **Examples**: `examples/*`
- **Minimum Rust version**: 1.77.2

---

## Build, Lint, and Test Commands

### Setup

```bash
pnpm install
pnpm build
```

### Rust Commands (Cargo)

**Run all tests:**

```bash
cargo test --workspace
```

**Run tests for a specific crate:**

```bash
cargo test -p tauri
cargo test -p tauri-utils
```

**Run a single test:**

```bash
cargo test test_name_here
cargo test --package tauri --lib test_name_here
cargo test --manifest-path crates/tauri/Cargo.toml test_name
```

**Run integration tests:**

```bash
cargo test --manifest-path crates/tests/acl/Cargo.toml
cargo test --manifest-path crates/tests/restart/Cargo.toml
```

**Run clippy lints:**

```bash
cargo clippy --all-targets --all-features -- -D warnings
```

**Check formatting:**

```bash
cargo fmt --all -- --check
```

**Build documentation:**

```bash
cargo +nightly doc --all-features --open
```

**Run a Tauri example:**

```bash
cargo run --example helloworld
```

### JavaScript/TypeScript Commands (PNPM)

**Build packages:**

```bash
pnpm build
pnpm build:api
pnpm build:cli
```

**Run tests:**

```bash
pnpm test
pnpm --filter "@tauri-apps/cli" test   # CLI JS tests
```

**Lint and format:**

```bash
pnpm format           # Format with Prettier
pnpm format:check     # Check Prettier formatting
pnpm eslint:check     # ESLint check
pnpm ts:check         # TypeScript type checking
```

**Run API example in dev mode:**

```bash
pnpm example:api:dev
```

---

## Code Style Guidelines

### General

- **License header required** on all new files:
  ```rust
  // Copyright 2019-2024 Tauri Programme within The Commons Conservancy
  // SPDX-License-Identifier: Apache-2.0
  // SPDX-License-Identifier: MIT
  ```

### Rust

- **Edition**: Rust 2021
- **Formatting**: See `rustfmt.toml` (max_width = 100, 2 spaces, Unix newlines)
- **Lints**: Enable `#![warn(rust_2018_idioms)]` in crates
- **Naming**:
  - Functions/variables: `snake_case`
  - Types/traits: `PascalCase`
  - Constants: `SCREAMING_SNAKE_CASE`
- **Imports**: Use `use` statements with grouped imports:
  ```rust
  use std::{
    path::Path,
    fs::File,
  };
  ```
- **Error handling**:
  - Use `thiserror` for custom error types with `#[derive(Debug, thiserror::Error)]`
  - Use `anyhow` for context-enabled errors (`anyhow::Context`, `anyhow::bail!`)
  - Use `anyhow::Result<T>` for functions that need rich error context
- **Serialization**: Use `serde` with `#[derive(Serialize, Deserialize)]`
- **Documentation**: Add doc comments (`///`) for public APIs; include platform-specific notes

### TypeScript/JavaScript

- **Formatting**: Prettier (2 space indent, single quotes)
- **Linting**: ESLint with TypeScript support
- **Type checking**: TypeScript strict mode
- **Naming**: camelCase for variables/functions, PascalCase for types/classes

### EditorConfig

All files follow `.editorconfig`:

- charset: utf-8
- indent_style: space
- indent_size: 2
- end_of_line: lf
- insert_final_newline: true
- trim_trailing_whitespace: true

---

## Code Organization

### Crates Structure

| Crate                      | Purpose                      |
| -------------------------- | ---------------------------- |
| `crates/tauri`             | Core framework               |
| `crates/tauri-build`       | Build-time codegen           |
| `crates/tauri-codegen`     | Compile-time code generation |
| `crates/tauri-macros`      | Procedural macros            |
| `crates/tauri-runtime`     | WebView runtime abstraction  |
| `crates/tauri-runtime-wry` | WRY runtime implementation   |
| `crates/tauri-utils`       | Shared utilities             |
| `crates/tauri-cli`         | Rust CLI                     |
| `crates/tauri-bundler`     | App bundler                  |
| `packages/api`             | TypeScript API               |
| `packages/cli`             | Node.js CLI wrapper          |

### Testing Guidelines

- Unit tests: Inline in `#[cfg(test)]` modules
- Integration tests: `crates/tests/*` directory
- Example apps: `examples/*` for manual testing

---

## Common Development Tasks

### Test local changes against an app

In your app's `src-tauri/Cargo.toml`:

```toml
tauri = { path = "path/to/local/tauri/crates/tauri" }
```

### Test CLI changes

```bash
cargo install --path crates/tauri-cli --debug
cargo tauri build
```

### Add a new crate

1. Create crate in `crates/`
2. Add to `Cargo.toml` workspace members
3. Add appropriate workflow triggers in `.github/workflows/`
