# Rust Stack Reference

## Detection Files
- `Cargo.toml` (primary)
- `Cargo.lock`

## Public vs Private API

### Visibility Modifiers
```rust
// Public to all
pub fn public_function() {}
pub struct PublicStruct {}

// Public within crate only
pub(crate) fn crate_internal() {}

// Public to parent module
pub(super) fn parent_visible() {}

// Private (default)
fn private_function() {}
struct PrivateStruct {}
```

### Module Re-exports
```rust
// src/lib.rs
pub mod api;           // Entire module is public
mod internal;          // Private module
pub use api::Client;   // Re-export specific items
```

### Struct Field Visibility
```rust
pub struct Config {
    pub host: String,      // Public field
    pub(crate) port: u16,  // Crate-internal
    password: String,      // Private
}
```

## Documentation Comments

### Doc Comments (///)
```rust
/// Creates a new client with the given configuration.
///
/// This function initializes a connection pool and validates
/// the configuration before returning.
///
/// # Arguments
///
/// * `config` - The client configuration
///
/// # Returns
///
/// A new `Client` instance, or an error if initialization fails.
///
/// # Errors
///
/// Returns `ConfigError` if the configuration is invalid.
/// Returns `ConnectionError` if unable to connect.
///
/// # Examples
///
/// ```
/// let config = Config::default();
/// let client = create_client(config)?;
/// ```
///
/// # Panics
///
/// Panics if the runtime is not available.
pub fn create_client(config: Config) -> Result<Client, Error>
```

### Module Documentation (//!)
```rust
//! Authentication module for user management.
//!
//! This module provides functions for authenticating users
//! and managing sessions.
//!
//! # Example
//!
//! ```
//! use myapp::auth;
//! let token = auth::login("user", "pass")?;
//! ```
```

### Common Doc Sections
- `# Examples` - Usage examples (tested by cargo test)
- `# Errors` - Possible error conditions
- `# Panics` - Conditions that cause panics
- `# Safety` - For unsafe functions
- `# Arguments` - Parameter descriptions
- `# Returns` - Return value description

## Error Handling Patterns

### Result Type
```rust
pub fn process(input: &str) -> Result<Output, ProcessError> {
    // ...
}
```

### Custom Error Types
```rust
/// Errors that can occur during authentication.
#[derive(Debug, thiserror::Error)]
pub enum AuthError {
    /// The provided credentials were invalid.
    #[error("invalid credentials")]
    InvalidCredentials,

    /// The user account is locked.
    #[error("account locked until {unlock_time}")]
    AccountLocked { unlock_time: DateTime<Utc> },

    /// An internal error occurred.
    #[error("internal error: {0}")]
    Internal(#[from] anyhow::Error),
}
```

### Error Conversion
```rust
impl From<IoError> for AppError {
    fn from(err: IoError) -> Self {
        AppError::Io(err)
    }
}
```

## Type System

### Traits
```rust
/// Types that can be serialized to JSON.
pub trait ToJson {
    /// Serialize this value to a JSON string.
    fn to_json(&self) -> Result<String, JsonError>;
}
```

### Generics and Bounds
```rust
pub fn process<T, E>(items: Vec<T>) -> Result<Vec<T>, E>
where
    T: Clone + Send + 'static,
    E: std::error::Error,
```

### Associated Types
```rust
pub trait Iterator {
    type Item;
    fn next(&mut self) -> Option<Self::Item>;
}
```

### Type Aliases
```rust
pub type Result<T> = std::result::Result<T, AppError>;
pub type BoxFuture<T> = Pin<Box<dyn Future<Output = T> + Send>>;
```

## Project Structure Conventions

### Library Crate
```
src/
  lib.rs          # Crate root, public API
  error.rs        # Error types
  types.rs        # Shared types
  api/
    mod.rs        # Module declarations
    client.rs
    request.rs
```

### Binary Crate
```
src/
  main.rs         # Entry point
  lib.rs          # Optional library portion
  cli.rs          # CLI handling
  config.rs       # Configuration
```

### Workspace
```
Cargo.toml        # Workspace definition
crates/
  core/           # Shared core functionality
  api/            # API types
  cli/            # CLI binary
```

## Common Patterns to Recognize

### Builder Pattern
```rust
pub struct ClientBuilder {
    host: Option<String>,
    port: Option<u16>,
}

impl ClientBuilder {
    pub fn new() -> Self { ... }
    pub fn host(mut self, host: &str) -> Self { ... }
    pub fn port(mut self, port: u16) -> Self { ... }
    pub fn build(self) -> Result<Client, BuildError> { ... }
}
```

### Newtype Pattern
```rust
pub struct UserId(pub String);
pub struct Email(String);  // Private inner value
```

### From/Into Implementations
```rust
impl From<&str> for UserId {
    fn from(s: &str) -> Self {
        UserId(s.to_string())
    }
}
```

### Default Implementation
```rust
impl Default for Config {
    fn default() -> Self {
        Config {
            timeout: Duration::from_secs(30),
            retries: 3,
        }
    }
}
```

### Async Functions
```rust
pub async fn fetch_data(url: &str) -> Result<Data, FetchError>
```
Document cancellation safety and required runtime.

### Feature Flags
```rust
#[cfg(feature = "json")]
pub fn to_json(&self) -> String { ... }
```
Document feature-gated functionality.
