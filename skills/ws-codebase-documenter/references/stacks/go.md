# Go Stack Reference

## Detection Files
- `go.mod` (primary)
- `go.sum`

## Public vs Private API

### Capitalization Rule
```go
// Public: Uppercase first letter
func PublicFunction() {}
type PublicStruct struct {}
const PublicConst = 1
var PublicVar int

// Private: Lowercase first letter
func privateHelper() {}
type internalStruct struct {}
```

### Package Visibility
- `internal/` directory: Only accessible by parent and sibling packages
- Items in `internal/` are never public API

### Exported Fields
```go
type Config struct {
    Host     string  // Public: uppercase
    Port     int     // Public: uppercase
    password string  // Private: lowercase
}
```

## Documentation Comments

### Package Comments
```go
// Package auth provides authentication and authorization
// functionality for the application.
//
// The main entry points are [Authenticate] and [Authorize].
package auth
```

### Function Comments
```go
// Authenticate verifies user credentials and returns a session token.
//
// It accepts a username and password, validates them against the
// configured authentication backend, and returns a JWT token on success.
//
// Example:
//
//	token, err := auth.Authenticate("user@example.com", "password123")
//	if err != nil {
//	    log.Fatal(err)
//	}
//	fmt.Println(token)
func Authenticate(username, password string) (string, error)
```

### Type Comments
```go
// User represents an authenticated user in the system.
//
// Users are created through [CreateUser] and can be retrieved
// with [GetUser] or [FindUsers].
type User struct {
    // ID is the unique identifier for the user.
    ID string

    // Email is the user's email address, used for login.
    Email string

    // CreatedAt is when the user account was created.
    CreatedAt time.Time
}
```

### Deprecated Items
```go
// Deprecated: Use NewClient instead.
func CreateClient() *Client
```

## Error Handling Patterns

### Custom Errors
```go
// Sentinel errors
var (
    ErrNotFound     = errors.New("not found")
    ErrUnauthorized = errors.New("unauthorized")
)

// Error types
type ValidationError struct {
    Field   string
    Message string
}

func (e *ValidationError) Error() string {
    return fmt.Sprintf("%s: %s", e.Field, e.Message)
}
```

### Error Wrapping
```go
if err != nil {
    return fmt.Errorf("failed to process user %s: %w", userID, err)
}
```

### Error Checking
```go
// errors.Is for sentinel errors
if errors.Is(err, ErrNotFound) { ... }

// errors.As for error types
var valErr *ValidationError
if errors.As(err, &valErr) { ... }
```

## Type System

### Interfaces
```go
// Reader is the interface for reading data.
type Reader interface {
    // Read reads up to len(p) bytes into p.
    Read(p []byte) (n int, err error)
}
```

### Type Aliases and Definitions
```go
type UserID string           // New type
type Handler = func() error  // Alias
```

### Generics (Go 1.18+)
```go
type Result[T any] struct {
    Value T
    Error error
}

func Map[T, U any](items []T, fn func(T) U) []U
```

## Project Structure Conventions

### Standard Layout
```
cmd/
  myapp/
    main.go           # Entry point
pkg/
  mypackage/          # Public packages
    mypackage.go
    types.go
internal/
  helper/             # Private packages
    helper.go
```

### Flat Layout (Small Projects)
```
main.go
handler.go
model.go
repository.go
```

### Package Organization
```go
// Single-file packages: package name matches directory
// Multi-file packages: all files share package declaration
// doc.go: Package-level documentation
```

## Common Patterns to Recognize

### Options Pattern
```go
type Option func(*Config)

func WithTimeout(d time.Duration) Option {
    return func(c *Config) {
        c.Timeout = d
    }
}

func NewClient(opts ...Option) *Client
```

### Constructor Functions
```go
func NewUser(name, email string) (*User, error)
func MustNewUser(name, email string) *User  // Panics on error
```

### Interface Satisfaction
```go
// Compile-time interface check
var _ io.Reader = (*MyReader)(nil)
```

### Context Usage
```go
func (s *Service) Process(ctx context.Context, req *Request) (*Response, error)
```
Document context cancellation and timeout behavior.

### Embedded Types
```go
type Server struct {
    http.Server  // Embedded, inherits methods
    logger *Logger
}
```
Document which methods come from embedded types.
