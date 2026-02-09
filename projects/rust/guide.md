# Rust Projects with Claude Code

## Tooling

| Tool | Purpose | Install |
|------|---------|---------|
| **cargo** | Build system and package manager | Included with [rustup](https://rustup.rs/) |
| **rustfmt** | Formatter | `rustup component add rustfmt` |
| **clippy** | Linter | `rustup component add clippy` |
| **cargo test** | Test runner (built-in) | Included with Cargo |
| **cargo-watch** | File watcher for dev loop | `cargo install cargo-watch` |

## Project Structure

### Binary

```
myapp/
├── src/
│   ├── main.rs
│   ├── lib.rs        (optional shared logic)
│   └── config.rs
├── tests/
│   └── integration.rs
├── Cargo.toml
└── CLAUDE.md
```

### Library

```
mylib/
├── src/
│   ├── lib.rs
│   └── utils.rs
├── tests/
│   └── integration.rs
├── Cargo.toml
└── CLAUDE.md
```

### Workspace (multiple crates)

```
myworkspace/
├── crates/
│   ├── core/
│   │   └── src/lib.rs
│   └── cli/
│       └── src/main.rs
├── Cargo.toml        (workspace root)
└── CLAUDE.md
```

## Style Conventions

- Follow the **Rust API Guidelines** and standard Rust naming
- Types are `PascalCase`, functions and variables are `snake_case`
- Constants are `SCREAMING_SNAKE_CASE`
- Use `Result<T, E>` for fallible operations, `Option<T>` for optional values
- Prefer returning `Result` over panicking
- Use `self` methods over free functions when operating on a type

```rust
use std::path::Path;

/// Loads configuration from a TOML file.
pub fn load_config(path: &Path) -> Result<Config, ConfigError> {
    let content = std::fs::read_to_string(path)
        .map_err(|e| ConfigError::ReadFailed(e))?;
    let config: Config = toml::from_str(&content)
        .map_err(|e| ConfigError::ParseFailed(e))?;
    Ok(config)
}
```

## Error Handling

Use `thiserror` for library errors and `anyhow` for application errors:

```rust
// Library error (thiserror)
use thiserror::Error;

#[derive(Error, Debug)]
pub enum AppError {
    #[error("database query failed")]
    Database(#[from] sqlx::Error),
    #[error("user {0} not found")]
    NotFound(String),
}

// Application error (anyhow)
use anyhow::{Context, Result};

fn main() -> Result<()> {
    let config = load_config("config.toml")
        .context("failed to load configuration")?;
    Ok(())
}
```

## Testing

### Unit tests (same file)

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_add() {
        assert_eq!(add(2, 3), 5);
    }

    #[test]
    fn test_parse_error() {
        let result = parse("invalid");
        assert!(result.is_err());
    }
}
```

### Integration tests (tests/ directory)

```rust
// tests/integration.rs
use mylib::process;

#[test]
fn test_full_pipeline() {
    let input = "test data";
    let result = process(input).unwrap();
    assert_eq!(result.status, "complete");
}
```

## Common Commands

```bash
cargo build                   # build debug
cargo build --release         # build optimized
cargo test                    # run all tests
cargo test test_name          # run specific test
cargo test -- --nocapture     # show println! output in tests
cargo clippy                  # lint
cargo fmt                     # format
cargo check                   # type-check without building
cargo doc --open              # generate and view docs
cargo watch -x test           # re-run tests on file change
```

## Dependencies

Managed via `Cargo.toml`:

```toml
[package]
name = "myapp"
version = "0.1.0"
edition = "2021"

[dependencies]
serde = { version = "1", features = ["derive"] }
tokio = { version = "1", features = ["full"] }
anyhow = "1"

[dev-dependencies]
assert_cmd = "2"
```

```bash
cargo add serde --features derive   # add dependency
cargo remove serde                  # remove dependency
cargo update                        # update all dependencies
```

## Common Crates

| Crate | Purpose |
|-------|---------|
| **serde** + **serde_json** | Serialization/deserialization |
| **tokio** | Async runtime |
| **anyhow** | Application error handling |
| **thiserror** | Library error types |
| **clap** | CLI argument parsing |
| **tracing** | Structured logging |
| **reqwest** | HTTP client |
| **sqlx** | Async SQL (Postgres, SQLite, MySQL) |
| **axum** | Web framework |

## Common Patterns to Suggest to Claude

- **Result/Option** chaining with `?`, `map`, `and_then`
- **Builder pattern** for complex struct construction
- **thiserror** for library error types, **anyhow** for applications
- **derive macros** (Debug, Clone, Serialize) to reduce boilerplate
- **trait objects** (`Box<dyn Trait>`) or generics for polymorphism
- **iterators** over manual loops for collection processing
