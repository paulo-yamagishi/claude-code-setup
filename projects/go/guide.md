# Go Projects with Claude Code

## Tooling

| Tool | Purpose | Install |
|------|---------|---------|
| **go** | Compiler and build tool | [go.dev/dl](https://go.dev/dl/) |
| **gofmt** | Formatter (built-in) | Included with Go |
| **golangci-lint** | Meta-linter (runs 50+ linters) | `go install github.com/golangci-lint/golangci-lint/cmd/golangci-lint@latest` |
| **go test** | Test runner (built-in) | Included with Go |
| **delve** | Debugger | `go install github.com/go-delve/delve/cmd/dlv@latest` |

## Project Structure

### Application (cmd/pkg/internal layout)

```
myproject/
├── cmd/
│   └── myapp/
│       └── main.go
├── internal/
│   ├── handler/
│   │   └── handler.go
│   └── service/
│       └── service.go
├── pkg/
│   └── client/
│       └── client.go
├── go.mod
├── go.sum
└── CLAUDE.md
```

- `cmd/` — Entry points (one subdirectory per binary)
- `internal/` — Private packages (cannot be imported by other modules)
- `pkg/` — Public packages (importable by other modules)

### Library

```
mylib/
├── mylib.go
├── mylib_test.go
├── go.mod
└── CLAUDE.md
```

## Style Conventions

- Follow **Effective Go** and the Go wiki code review comments
- Exported names are `PascalCase`, unexported are `camelCase`
- Acronyms are all caps: `HTTPClient`, `userID`
- Always handle errors — never use `_` for error returns
- Keep functions short and focused
- Use `context.Context` as the first parameter for functions that do I/O

```go
// UserByID fetches a user by their unique identifier.
func (s *Service) UserByID(ctx context.Context, id string) (*User, error) {
    user, err := s.repo.Find(ctx, id)
    if err != nil {
        return nil, fmt.Errorf("finding user %s: %w", id, err)
    }
    return user, nil
}
```

## Error Handling

```go
// Wrap errors with context using fmt.Errorf and %w
if err != nil {
    return fmt.Errorf("opening config file: %w", err)
}

// Define sentinel errors for expected conditions
var ErrNotFound = errors.New("not found")

// Check sentinel errors with errors.Is
if errors.Is(err, ErrNotFound) {
    // handle missing resource
}
```

## Testing

Go uses **table-driven tests** as the standard pattern:

```go
func TestAdd(t *testing.T) {
    tests := []struct {
        name     string
        a, b     int
        expected int
    }{
        {"positives", 1, 2, 3},
        {"zeros", 0, 0, 0},
        {"negative", -1, 1, 0},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got := Add(tt.a, tt.b)
            if got != tt.expected {
                t.Errorf("Add(%d, %d) = %d, want %d", tt.a, tt.b, got, tt.expected)
            }
        })
    }
}
```

Use `testify` for assertion helpers when standard `if` checks become verbose:

```go
import "github.com/stretchr/testify/assert"

func TestUser(t *testing.T) {
    user, err := GetUser("123")
    assert.NoError(t, err)
    assert.Equal(t, "Alice", user.Name)
}
```

## Common Commands

```bash
go build ./...                # build all packages
go test ./...                 # run all tests
go test -v ./...              # verbose test output
go test -run TestName ./pkg/  # run specific test
go test -race ./...           # detect race conditions
go vet ./...                  # static analysis
golangci-lint run             # run all linters
gofmt -w .                   # format all files
```

## Dependencies

Managed via `go.mod`:

```
module github.com/user/myproject

go 1.22

require (
    github.com/gin-gonic/gin v1.9.1
    github.com/stretchr/testify v1.8.4
)
```

```bash
go mod tidy       # add missing, remove unused dependencies
go mod download   # download dependencies to cache
go get pkg@v1.2   # add or update a dependency
```

## Common Patterns to Suggest to Claude

- **interfaces** for dependency injection and testability
- **table-driven tests** for comprehensive test coverage
- **error wrapping** with `fmt.Errorf` and `%w` for debuggable errors
- **context.Context** for cancellation and timeouts
- **sync.WaitGroup** or **errgroup** for concurrent operations
- **embed** for bundling static assets
