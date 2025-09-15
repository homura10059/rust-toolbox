# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Rust monorepo (workspace) containing various tools and utilities. The workspace allows multiple related Rust projects to be developed together while sharing dependencies and build configuration.

## Workspace Structure

```
rust-toolbox/
├── Cargo.toml          # Workspace configuration
├── crates/             # Individual tool crates
│   ├── tool1/
│   │   ├── Cargo.toml
│   │   └── src/
│   └── tool2/
│       ├── Cargo.toml
│       └── src/
└── target/             # Shared build artifacts
```

## Common Commands

```bash
# Build all workspace members
cargo build

# Build specific crate
cargo build -p <crate-name>

# Run specific binary
cargo run -p <crate-name>

# Run all tests in workspace
cargo test

# Test specific crate
cargo test -p <crate-name>

# Check all workspace members
cargo check

# Format all code in workspace
cargo fmt

# Run clippy on all members
cargo clippy

# Build for release
cargo build --release
```

## Adding New Tools

To add a new tool to the workspace:

1. Create a new crate:
   ```bash
   cargo new crates/<tool-name> --bin
   ```

2. Add the new crate to the workspace `Cargo.toml` members array:
   ```toml
   members = [
       "crates/<tool-name>",
   ]
   ```

## Development Notes

- Workspace dependencies are defined in the root `Cargo.toml` for sharing across members
- Each tool can have its own dependencies in addition to workspace dependencies
- The workspace uses Rust 2021 edition
- Common dependencies like `serde`, `clap`, `tokio` can be added to workspace.dependencies as needed