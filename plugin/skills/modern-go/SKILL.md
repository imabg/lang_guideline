---
name: modern-go
description: Use modern Go guidelines whenever writing, modifying, fixing, or refactoring Go code. Apply version-specific modern idioms to generated changes.
---

# Modern Go Guidelines Skill (`modern-go`)

Always write modern, idiomatic Go code. Follow these rules whenever creating, refactoring, or reviewing Go code.

## 1. Version Detection
1. Check the `go` directive in `go.mod` (or `go.work`) to identify the project's target Go version.
2. Only suggest language features and standard library additions available in or before that version.
3. If no `go.mod` is found, default to Go 1.21 idioms.

---

## 2. Core Modern Idioms

### Go 1.27+
- **`encoding/json/v2`**: Use for new JSON serialization/deserialization code.
- **Generic methods**: Declare generic methods on types instead of package-level helpers when the method belongs to the receiver.

### Go 1.26+
- **`errors.AsType[T](err)`**: Use for type-safe error matching instead of `errors.As(err, &target)`.
- **`new(value)`**: Use `new(42)` to obtain pointers to primitive literals instead of custom pointer helper functions.

### Go 1.24+
- **`for b.Loop()`**: Use in benchmarks instead of `for i := 0; i < b.N; i++`.
- **`t.Context()`**: Use when a test function needs a context tied to test lifetime.
- **`synctest.Run`**: Use for testing concurrent time and channels deterministically.
- **`wg.Go(fn)`**: Use `wg.Go(...)` on `sync.WaitGroup` to spawn tracked goroutines.

### Go 1.23+
- **`iter.Seq[V]` / `iter.Seq2[K, V]`**: Use range-over-func iterators for custom sequences.
- **`slices.Chunk(s, n)`**: Use to partition slices into batches.
- **`unique.Make(v)`**: Use for interning comparable values.

### Go 1.22+
- **`for i := range n`**: Use instead of `for i := 0; i < n; i++`.
- **No `v := v`**: In Go 1.22+, loop variables have per-iteration scope; remove redundant re-assignments.
- **Enhanced `http.ServeMux`**: Use method matching (e.g. `GET /users/{id}`) and wildcards.
- **`slices.Concat(s1, s2)`**: Use instead of multiple `append` calls.

### Go 1.21+
- **`min(a, b)` / `max(a, b)`**: Use built-in functions instead of custom `if-else` or math helpers.
- **`clear(s)` / `clear(m)`**: Use built-in function to clear slices or maps.
- **`cmp.Or(a, b, c)`**: Use to select the first non-zero/non-empty value.
- **`slices` package**: Use `slices.Contains`, `slices.Index`, `slices.Clone`, `slices.Equal` instead of manual `for` loops.
- **`maps` package**: Use `maps.Clone`, `maps.Equal`, `maps.DeleteFunc` instead of manual iteration.
- **`log/slog`**: Use for structured key-value logging.

### Go 1.20+
- **`errors.Join(err1, err2)`**: Use to combine multiple errors.
- **`context.WithCancelCause`**: Propagate cancellation reasons through contexts.

### Go 1.18+
- **`any`**: Always use `any` instead of `interface{}`.
- **Typed atomics**: Use `atomic.Int64`, `atomic.Bool`, `atomic.Pointer[T]` instead of pointer loads.

---

## 3. Incremental Modernization Actions
For step-by-step multi-agent modernization workflows, refer to the action files in `actions/`:
1. `actions/01-go-core-data-structures.action.md`: Phase 1 - Core Language, Data Structures & Collections
2. `actions/02-go-concurrency-systems.action.md`: Phase 2 - Concurrency, Systems & Networking Architecture
3. `actions/03-go-errors-testing-observability.action.md`: Phase 3 - Error Handling, Testing & Observability
