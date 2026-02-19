# Tauri Agent Development Guide

This guide is for agentic coding assistants working in the Tauri repository.

## Project Overview

Tauri is a polyglot framework for building tiny, fast desktop applications using Rust + HTML/CSS/JS.

- **Rust workspace**: Multiple crates in `crates/` (tauri, tauri-runtime, tauri-utils, tauri-macros, etc.)
- **JavaScript/TypeScript**: API package in `packages/api/` with TypeScript definitions
- **Package manager**: pnpm (workspace-based, see `pnpm-workspace.yaml`)
- **Rust version**: 1.77.2 minimum (see `Cargo.toml`)

## Build Commands

### Rust (Workspace)

```bash
# Build all crates
cargo build --workspace

# Build specific crate
cargo build -p tauri
cargo build -p tauri-cli

# Build with all features
cargo build --all-features

# Build without default features
cargo build --no-default-features

# Release build (optimized, see profile in Cargo.toml)
cargo build --release

# Run example (automatically rebuilds local crates)
cargo run --example helloworld
```

### JavaScript/TypeScript

```bash
# Install dependencies (from root)
pnpm install

# Build all packages
pnpm build

# Build in debug mode
pnpm build:debug

# Build specific package
pnpm build:api
pnpm build:cli

# Type check all packages
pnpm ts:check
```

## Testing Commands

### Rust Tests

```bash
# Run all tests in workspace
cargo test --workspace

# Run tests for specific crate
cargo test -p tauri
cargo test -p tauri-utils

# Run specific test by name
cargo test test_name

# Run tests matching pattern
cargo test pattern_name

# Run single test file (use module path)
cargo test --lib module_name::test_name

# Run tests with all features
cargo test --all-features

# Run tests without default features
cargo test --no-default-features

# Important: Set environment variable for tests (already in .cargo/config.toml)
__TAURI_WORKSPACE__=true cargo test
```

### JavaScript/TypeScript Tests

```bash
# Run all tests
pnpm test

# Run tests for specific package
pnpm run --filter "@tauri-apps/api" test
```

## Linting & Formatting

### Rust

```bash
# Format code (max_width=100, 2 spaces, see rustfmt.toml)
cargo fmt --all

# Check formatting without modifying
cargo fmt --all -- --check

# Run clippy (lint) - MUST pass with no warnings
cargo clippy --all-targets --all-features -- -D warnings

# Format TOML files
taplo fmt
```

### JavaScript/TypeScript

```bash
# Format with prettier
pnpm format

# Check formatting
pnpm format:check

# Run ESLint check
pnpm eslint:check

# Fix ESLint issues
pnpm run --filter "@tauri-apps/api" eslint:fix
```

## Code Style Guidelines

### Rust

#### Formatting (rustfmt.toml)

- **Line width**: 100 characters maximum
- **Indentation**: 2 spaces (not tabs)
- **Imports**: Reordered and grouped automatically
- **Line endings**: Unix (LF)
- **Edition**: 2021

#### Imports

```rust
// Standard library first
use std::sync::Arc;
use std::path::Path;

// External crates
use serde::{Deserialize, Serialize};
use tokio::sync::Mutex;

// Crate imports
use crate::manager::Manager;
use crate::{App, Runtime};

// Module re-exports at top of file
pub use error::{Error, Result};
```

#### Naming Conventions

- **Types/Traits**: PascalCase (`AppHandle`, `Runtime`)
- **Functions/Methods**: snake_case (`invoke_handler`, `build_app`)
- **Constants**: SCREAMING_SNAKE_CASE (`DEFAULT_WINDOW_TITLE`)
- **Modules**: snake_case (in file/folder names)

#### Error Handling

- Use `Result<T, Error>` with custom error types (see `crates/tauri/src/error.rs`)
- Prefer `thiserror` for error definitions
- Use `anyhow` for application errors where appropriate
- Document error conditions in doc comments

#### Types & Generics

- Use explicit type annotations for public APIs
- Generic bounds: prefer `where` clauses for readability
- Use `#[cfg_attr(docsrs, feature(doc_cfg))]` for feature-gated APIs

#### Documentation

- **Public items MUST have doc comments** (`///`)
- Start with brief summary line
- Use `# Examples` sections with runnable code
- Use `# Panics`, `# Errors`, `# Safety` sections as appropriate
- See `crates/tauri/src/lib.rs` for comprehensive examples

````rust
/// Creates a new window with the given label.
///
/// # Examples
///
/// ```rust
/// let window = app.create_window("main", WindowUrl::default())?;
/// ```
///
/// # Errors
///
/// Returns error if window with same label already exists.
pub fn create_window<R: Runtime>(...) -> Result<Window<R>>
````

### TypeScript/JavaScript (packages/api)

#### Formatting (.prettierrc)

- **Quotes**: Single quotes
- **Semicolons**: No semicolons
- **Trailing commas**: None
- **Line breaks**: Start operators on new line

#### TypeScript Config (tsconfig.json)

- **Target**: ES2019
- **Module**: esnext
- **Strict mode**: Enabled
- `noUnusedLocals: true`
- `noImplicitAny: true`

#### ESLint Rules (eslint.config.js)

- **No console/debugger** in production code
- **Security rules enabled**: detect-unsafe-regex, detect-eval-with-expression
- **Unused vars**: Prefix with `_` to ignore (enforced)
- TypeScript recommended + type-checked rules

#### Imports

```typescript
// Core/built-in imports first
import { invoke, addPluginListener } from './core'

// Type imports
import type { PluginListener } from './core'
import { Image } from './image'
```

#### Documentation

- Use JSDoc comments for all exported functions/types
- Include `@param`, `@returns`, `@example` tags
- Document exceptions/errors with `@throws`

````typescript
/**
 * Gets the application version.
 *
 * @example
 * ```typescript
 * import { getVersion } from '@tauri-apps/api/app'
 * const version = await getVersion()
 * ```
 *
 * @returns The application version string.
 */
export async function getVersion(): Promise<string>
````

## File Headers

All source files MUST include copyright/license header:

```
// Copyright 2019-2024 Tauri Programme within The Commons Conservancy
// SPDX-License-Identifier: Apache-2.0
// SPDX-License-Identifier: MIT
```

## Common Patterns

### Rust: Command Handlers

```rust
#[tauri::command]
async fn my_command(
  app: AppHandle,
  window: Window,
  state: State<'_, MyState>,
) -> Result<String> {
  // Implementation
  Ok("result".into())
}
```

### TypeScript: Invoke Pattern

```typescript
export async function myFunction(): Promise<string> {
  return await invoke('plugin:namespace|my_command')
}
```

## Testing Patterns

### Rust: Unit Tests

```rust
#[cfg(test)]
mod tests {
  use super::*;

  #[test]
  fn test_something() {
    assert_eq!(result, expected);
  }

  #[tokio::test]
  async fn test_async_something() {
    // async test
  }
}
```

### Using Test Module

```rust
#[cfg(test)]
mod tests {
  use tauri::test::{mock_builder, mock_context};

  #[test]
  fn test_with_mock_app() {
    let app = mock_builder().build(mock_context()).unwrap();
    // test with mock app
  }
}
```

## Important Notes

1. **CI Requirements**: All tests, clippy, and formatting checks MUST pass
2. **Signed commits required**: Use `git config commit.gpgsign true`
3. **Change files**: Follow `.changes/readme.md` for PR changelog entries
4. **No `println!` in library code**: Use `log` crate or `tracing` feature
5. **Feature gates**: Many APIs are behind cargo features - check `Cargo.toml`
6. **Local testing**: Use `cargo run --example helloworld` to test core changes

## References

- Architecture: `ARCHITECTURE.md`
- Contributing: `.github/CONTRIBUTING.md`
- Rust docs: `cargo +nightly doc --all-features --open`
