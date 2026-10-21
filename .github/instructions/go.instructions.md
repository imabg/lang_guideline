---
applyTo: "**/*.go"
---

# Modern Go Development Guidelines

You are an expert Go developer writing modern, idiomatic, high-performance Go code. Follow these rules whenever writing, editing, or refactoring Go code.

---

## 1. Core Principles & Version Compliance

1. **Version-Gated Idioms**:
   - Always check the `go` directive in `go.mod` (or `go.work`) before suggesting language features or standard library APIs.
   - Do **NOT** use features newer than the project's declared Go version.
   - When a modern standard library function is available for the target Go version, **always prefer it over legacy code, manual loops, or third-party helper libraries**.

2. **Style & Idiom Fundamentals**:
   - **Happy Path Left-Aligned**: Return early on error or guard condition to eliminate nested `if-else` blocks.
   - **Error Handling**: Wrap errors contextually using `fmt.Errorf("doing something: %w", err)`. Never discard errors with `_ =` unless explicitly justified.
   - **Zero Values**: Design structs and types so that their zero-value is safe and ready to use without explicit initialization whenever possible.
   - **Standard Library First**: Do not introduce external dependencies for functionality provided by modern Go standard libraries (`slices`, `maps`, `cmp`, `errors`, `net/http`, `log/slog`).

---

## 2. Modern Go Idioms by Version

### Go 1.27+

#### `json_v2`: Use `encoding/json/v2` for new JSON code
- Prefer `encoding/json/v2` for new JSON serialization and deserialization. It features stricter, interoperable defaults, omits zero-values cleanly, and avoids historical quirks of v1.
```go
// Before (legacy)
import "encoding/json"

// After (Go 1.27+)
import "encoding/json/v2"
```

#### `generic_methods`: Use generic methods on types
- When a generic operation belongs naturally to a receiver type, declare it as a generic method instead of an external package-level helper function.
```go
// Before (legacy helper)
func Map[T comparable, U any](s Set[T], f func(T) U) []U { ... }

// After (Go 1.27+)
func (s Set[T]) Map[U any](f func(T) U) []U { ... }
```

---

### Go 1.26+

#### `errors_as_type`: Type-safe error unwrapping
- Use `errors.AsType[T](err)` instead of declaring an uninitialized target variable with `errors.As(err, &target)`.
```go
// Before (legacy)
var pathErr *fs.PathError
if errors.As(err, &pathErr) {
    log.Println(pathErr.Path)
}

// After (Go 1.26+)
if pathErr, ok := errors.AsType[*fs.PathError](err); ok {
    log.Println(pathErr.Path)
}
```

#### `new_expression`: Pointers to literal values
- Use `new(value)` to directly obtain a pointer to a primitive or literal value instead of allocating helper functions or local variables.
```go
// Before (legacy helper)
func ptr[T any](v T) *T { return &v }
req := Request{Timeout: ptr(30 * time.Second)}

// After (Go 1.26+)
req := Request{Timeout: new(30 * time.Second)}
```

---

### Go 1.24+

#### `testing_b_loop`: Modern benchmark loops
- Use `b.Loop()` in benchmarks instead of `for i := 0; i < b.N; i++`. It simplifies setup/teardown and avoids benchmark timer resets.
```go
// Before (legacy)
func BenchmarkProcess(b *testing.B) {
    setup()
    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        Process()
    }
}

// After (Go 1.24+)
func BenchmarkProcess(b *testing.B) {
    setup()
    for b.Loop() {
        Process()
    }
}
```

#### `testing_t_context`: Test context lifetime
- Use `t.Context()` when a test needs a `context.Context` tied to the test execution lifetime instead of manually creating `context.WithCancel`.
```go
// Before (legacy)
ctx, cancel := context.WithCancel(context.Background())
defer cancel()
res, err := client.Fetch(ctx)

// After (Go 1.24+)
res, err := client.Fetch(t.Context())
```

#### `synctest`: Deterministic concurrent time testing
- Use `synctest.Run` to test concurrent code that involves channels, timeouts, and `time.Sleep` without slow real-world clocks.
```go
// After (Go 1.24+)
synctest.Run(func() {
    go delayedWorker()
    synctest.Wait()
})
```

#### `sync_waitgroup_go`: Spawning goroutines with WaitGroup
- Use `wg.Go(fn)` instead of manually pairing `wg.Add(1)` with `defer wg.Done()` around a goroutine.
```go
// Before (legacy)
wg.Add(1)
go func() {
    defer wg.Done()
    doWork()
}()

// After (Go 1.24+)
wg.Go(doWork)
```

#### `os_root`: Directory-scoped file operations
- Use `os.OpenRoot(dir)` to restrict filesystem operations to a specific root directory and prevent path traversal vulnerabilities.
```go
// After (Go 1.24+)
root, err := os.OpenRoot("/secure/dir")
if err != nil { return err }
defer root.Close()
f, err := root.Open("file.txt")
```

---

### Go 1.23+

#### `iterators`: Standard range-over-func iterators
- Implement and consume custom iterators using `iter.Seq[V]` (single-value) and `iter.Seq2[K, V]` (key-value) instead of returning slices or using channel streaming.
```go
// After (Go 1.23+)
import "iter"

func (tree *Tree) All() iter.Seq[string] {
    return func(yield func(string) bool) {
        tree.walk(func(val string) bool {
            return yield(val)
        })
    }
}

// Usage:
for val := range tree.All() {
    fmt.Println(val)
}
```

#### `slices_chunk`: Chunking slices
- Use `slices.Chunk(slice, n)` to divide a slice into chunks instead of calculating manual loop bounds.
```go
// After (Go 1.23+)
for chunk := range slices.Chunk(users, 50) {
    processBatch(chunk)
}
```

#### `unique_make`: Canonical value interning
- Use `unique.Make(v)` for interning comparable values (e.g. repeated strings or network headers) to save memory and enable fast pointer comparisons.

---

### Go 1.22+

#### `for_range_int`: Range over integers
- Use `for i := range n` instead of `for i := 0; i < n; i++` for simple count-based loops.
```go
// Before (legacy)
for i := 0; i < count; i++ {
    doAction(i)
}

// After (Go 1.22+)
for i := range count {
    doAction(i)
}
```

#### `loop_variable_scoping`: Per-iteration loop variable semantics
- In Go 1.22+, loop variables in `for` loops have per-iteration scope. Do **NOT** add redundant `v := v` reassignment lines inside loop bodies when spawning goroutines or closures.
```go
// Before (Go 1.21 and older)
for _, item := range items {
    item := item // Redundant in Go 1.22+
    go func() { process(item) }()
}

// After (Go 1.22+)
for _, item := range items {
    go func() { process(item) }()
}
```

#### `http_routing_patterns`: Enhanced `http.ServeMux`
- Use built-in HTTP method matching and path wildcards with `http.NewServeMux` instead of pulling in third-party routers like Chi or Gorilla Mux.
```go
// After (Go 1.22+)
mux := http.NewServeMux()
mux.HandleFunc("GET /users/{id}", getUserHandler)
mux.HandleFunc("POST /users", createUserHandler)

// Handler extraction:
id := r.PathValue("id")
```

#### `slices_concat`: Concatenating slices
- Use `slices.Concat(s1, s2, s3)` instead of chaining `append(append(s1, s2...), s3...)`.
```go
// Before (legacy)
combined := append(append([]string{}, first...), second...)

// After (Go 1.22+)
combined := slices.Concat(first, second)
```

---

### Go 1.21+

#### `builtin_min_max`: Use `min` and `max`
- Use the built-in `min(a, b)` and `max(a, b)` functions instead of custom `if-else` blocks or `math.Min`/`math.Max` (which require float64 casting).
```go
// Before (legacy)
func minInt(a, b int) int {
    if a < b { return a }
    return b
}

// After (Go 1.21+)
smallest := min(a, b, c)
```

#### `builtin_clear`: Clearing slices and maps
- Use built-in `clear(slice)` to zero all elements (preserving length and capacity) and `clear(map)` to remove all map entries.
```go
// Before (legacy map clear)
for k := range m {
    delete(m, k)
}

// After (Go 1.21+)
clear(m)
```

#### `cmp_or`: First non-zero value
- Use `cmp.Or(a, b, c)` to return the first non-zero/non-empty value instead of verbose `if-else` chains.
```go
// Before (legacy)
port := os.Getenv("PORT")
if port == "" {
    port = config.DefaultPort
}
if port == "" {
    port = "8080"
}

// After (Go 1.21+)
port := cmp.Or(os.Getenv("PORT"), config.DefaultPort, "8080")
```

#### `slices_package`: Standard slice manipulation
- Replace manual search, clone, and comparison loops with the `slices` package:
  - `slices.Contains(s, val)`: checks if element exists.
  - `slices.Index(s, val)` / `slices.IndexFunc(s, fn)`: finds index.
  - `slices.Clone(s)`: creates shallow copy.
  - `slices.Equal(s1, s2)`: compares equality.
  - `slices.Delete(s, i, j)`: removes a slice range.
```go
// Before (legacy manual loop)
found := false
for _, v := range users {
    if v == target {
        found = true
        break
    }
}

// After (Go 1.21+)
found := slices.Contains(users, target)
```

#### `maps_package`: Standard map operations
- Replace manual map copies, comparisons, and deletions with the `maps` package:
  - `maps.Clone(m)`
  - `maps.Equal(m1, m2)`
  - `maps.DeleteFunc(m, fn)`
```go
// Before (legacy manual copy)
dst := make(map[string]int, len(src))
for k, v := range src {
    dst[k] = v
}

// After (Go 1.21+)
dst := maps.Clone(src)
```

#### `log_slog`: Structured logging
- Use `log/slog` for structured, key-value logging rather than unformatted `log.Printf` or external logging frameworks unless one is already established in the codebase.
```go
// After (Go 1.21+)
slog.Info("processing order", "order_id", orderID, "amount", amount)
```

---

### Go 1.20+

#### `errors_join`: Combining multiple errors
- Use `errors.Join(err1, err2)` instead of third-party multierror packages.
```go
// Before (legacy custom wrapper)
type multiError []error
...

// After (Go 1.20+)
return errors.Join(errA, errB)
```

#### `context_with_cancel_cause`: Error causes in context
- Use `context.WithCancelCause` and `context.Cause(ctx)` to propagate the root reason why a context was canceled.
```go
// After (Go 1.20+)
ctx, cancel := context.WithCancelCause(parentCtx)
cancel(fmt.Errorf("database connection dropped"))

// Reader:
if err := context.Cause(ctx); err != nil {
    log.Println("cancellation cause:", err)
}
```

---

### Go 1.18+

#### `any_alias`: Use `any` instead of `interface{}`
- Always use `any` instead of `interface{}`.
```go
// Before (legacy)
func PrintValue(val interface{}) { ... }

// After (Go 1.18+)
func PrintValue(val any) { ... }
```

#### `atomic_typed`: Use typed atomic values
- Use `atomic.Int64`, `atomic.Uint64`, `atomic.Bool`, and `atomic.Pointer[T]` instead of passing pointers to `atomic.LoadInt64(&val)`.
```go
// Before (legacy)
var counter int64
atomic.AddInt64(&counter, 1)
val := atomic.LoadInt64(&counter)

// After (Go 1.18+)
var counter atomic.Int64
counter.Add(1)
val := counter.Load()
```

#### `netip_addr`: Modern IP addressing
- Prefer `netip.Addr` and `netip.Prefix` over `net.IP` for networking. `netip.Addr` is immutable, comparable, zero-allocation, and supports dual-stack IPv4/IPv6 seamlessly.
