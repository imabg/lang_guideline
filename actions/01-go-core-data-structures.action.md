---
name: "action-01-go-core-data-structures"
title: "Modern Go Core Language, Data Structures & Collections Modernization"
version: "1.1.0"
agent_role: "Modern Go Core & Collections Specialist Agent"
target_scope: "Core data models, slice/map algorithms, built-ins, iterations, pointers, and serialization"
target_languages: ["go"]
phase: 1
handover_input: "Phase 0 initialized workspace (actions/00-project-setup.action.md) or existing Go repository"
handover_output: "Modernized data models, collections, iterators, zeroing logic, passing go vet and compilation"
next_action: "actions/02-go-concurrency-systems.action.md"
---

# Action 01: Modern Go Core Language, Data Structures & Collections

## 1. Executive Mission & Agent Role

You are **Agent 1: Core & Collections Specialist**.
Your mission is to execute Phase 1 of building or modernizing a Go repository. You operate strictly on the foundational data structures, collection manipulation routines, language built-ins, iteration mechanics, pointers, and serialization models.

### Scope Boundaries
- **IN SCOPE**:
  - `go.mod` target version parsing, gate validation, and toolchain checks.
  - Slices manipulation (`slices` package: `Contains`, `Index`, `IndexFunc`, `Clone`, `Equal`, `Delete`, `Concat`, `Chunk`, `SortFunc`, `BinarySearch`).
  - Map manipulation (`maps` package: `Clone`, `Equal`, `DeleteFunc`, `Keys`, `Values`).
  - Memory wiping & zeroing (`clear(slice)`, `clear(map)`).
  - Conditionals and value fallback chains (`min()`, `max()`, `cmp.Or()`, `cmp.Compare()`).
  - Loops and traversal (`for i := range n`, removal of `v := v` loop closures, `iter.Seq` and `iter.Seq2` push iterators).
  - Primitive and literal pointers (`new(value)`).
  - Modern serialization (`encoding/json/v2` with `omitzero`).
  - Type simplification (`any` instead of `interface{}`, generic receiver methods).
- **OUT OF SCOPE** (Strictly deferred to downstream agents):
  - Goroutines, atomics, HTTP routing, and sandboxed I/O (*Handed to Agent 2*).
  - Test suites, benchmarks, synctest, and structured logging (*Handed to Agent 3*).

---

## 2. Model Decision Heuristics & Decision Matrices

To ensure AI models take the correct engineering decisions rather than blindly replacing code, apply these explicit decision matrices:

### Decision Matrix 1: Choosing Collection Lookup Strategy
When an agent encounters a slice lookup or containment check, follow this decision tree:

| Scenario / Constraints | Recommended Modern Pattern | Time Complexity | Allocations | Why / Decision Rationale |
| :--- | :--- | :--- | :--- | :--- |
| **Small slice (<30 items), unsorted, infrequent checks** | `slices.Contains(s, target)` | $O(N)$ | 0 | Cache-locality on small contiguous memory beats map hashing overhead. |
| **Slice is already sorted** | `slices.BinarySearch(s, target)` | $O(\log N)$ | 0 | Exponentially faster than linear scan; returns insertion index if absent. |
| **Frequent checks (>50 calls) on medium/large collection** | Convert to `map[K]struct{}` | $O(1)$ | 1 map allocation | Amortized $O(1)$ lookup prevents CPU bottlenecks in tight loops. |
| **Custom condition (e.g. matching struct field)** | `slices.IndexFunc(s, pred)` | $O(N)$ | 0 | Avoids writing manual loop counters and break statements. |

### Decision Matrix 2: Choosing Slice Clearing & Truncating Strategy
When an agent needs to empty or reset a slice:

| Goal / Context | Correct Go Pattern | Memory / GC Behavior | Common Model Mistake to Avoid |
| :--- | :--- | :--- | :--- |
| **Zero sensitive memory (tokens, passwords)** | `clear(s)` | Overwrites slots with zero values; `len` and `cap` preserved | `s = nil` does NOT zero memory in the underlying array! |
| **Reset length for buffer reuse without reallocating** | `s = s[:0]` | Preserves capacity; resets length | Retains underlying object pointers, delaying GC cleanup! |
| **Reset length AND allow GC of old object pointers** | `clear(s); s = s[:0]` | Zeroes object pointers for GC, then resets length | `clear` alone leaves length unchanged. |
| **Complete deallocation** | `s = nil` | Drops reference, allows GC of backing array | Using `clear` when full deallocation is desired. |

### Decision Matrix 3: Integer Loops & Iterators
When an agent encounters a counting loop:

| Context | Recommended Pattern | When NOT to use |
| :--- | :--- | :--- |
| **Iterating $0$ to $n-1$** | `for i := range n` (Go 1.22+) | Do not use if step size $\neq 1$ or decrementing (`i--`). |
| **Iterating $n$ times without using counter** | `for range n` (Go 1.22+) | Do not use if index `i` is needed inside loop body. |
| **Custom tree/graph/generator traversal** | `iter.Seq[T]` (Go 1.23+) | Do not use if returning a pre-existing in-memory slice is sufficient. |

### Decision Matrix 4: Fallback & Default Value Selection
| Context | Recommended Pattern | Anti-Pattern |
| :--- | :--- | :--- |
| **First non-zero value among strings/ints** | `cmp.Or(val, fallback, default)` | Nested `if val == "" { if fallback == "" ... }` |
| **Selecting bounding limits** | `min(max(val, low), high)` | Custom `clamp()` helper or `math.Min` float casts |

---

## 3. Version Compatibility Matrix

| Rule ID | Minimum Go | Modern Construct | Legacy Construct |
| :--- | :--- | :--- | :--- |
| `any_alias` | Go 1.18+ | `any` | `interface{}` |
| `slices_package` | Go 1.21+ | `slices.Contains`, `Clone`, `IndexFunc` | Manual `for` loops, manual `make`+`copy` |
| `maps_package` | Go 1.21+ | `maps.Clone`, `Equal`, `DeleteFunc` | Manual map iteration loops |
| `builtin_min_max` | Go 1.21+ | `min(a, b)`, `max(a, b)` | Custom ternary-style functions or `math.Min` |
| `builtin_clear` | Go 1.21+ | `clear(slice)`, `clear(map)` | Manual delete loops, manual zero loops |
| `cmp_or` | Go 1.21+ | `cmp.Or(a, b, c)` | Cascading `if-else` empty checks |
| `for_range_int` | Go 1.22+ | `for i := range n` | `for i := 0; i < n; i++` |
| `loop_variable_scoping`| Go 1.22+ | Native per-iteration variable | Redundant `v := v` variable shadowing |
| `slices_concat` | Go 1.22+ | `slices.Concat(s1, s2, s3)` | Chained `append(append(s1, s2...), s3...)` |
| `iterators` | Go 1.23+ | `iter.Seq[V]`, `iter.Seq2[K, V]` | Slice allocations or channel streaming |
| `slices_chunk` | Go 1.23+ | `slices.Chunk(s, size)` | Manual loop bounds math: `i:min(i+size, len)` |
| `unique_make` | Go 1.23+ | `unique.Make(v)` | Manual string interning maps |
| `new_expression` | Go 1.26+ | `new(value)` | Helper `func ptr[T](v T) *T { return &v }` |
| `generic_methods` | Go 1.27+ | `func (r Receiver[T]) Method[U any]()` | Standalone functions `func Method[T, U](r ...)` |
| `json_v2` | Go 1.27+ | `encoding/json/v2` with `omitzero` | `encoding/json` with pointer `omitempty` |

---

## 4. Detailed Directives with Production-Grade Code Samples

---

### Directive 1.1: Standardize Dynamic Types (`any_alias`)
- **Version Gate**: Go 1.18+
- **Rationale**: `any` is a built-in alias for `interface{}`. It improves readability, eliminates noise, and standardizes Go generics and type assertions.
- **Model Rule**: Replace `interface{}` with `any`. Do NOT replace non-empty interfaces declaring methods (e.g., `interface{ String() string }`).

#### Code Transformation Sample
```go
// ============================================================================
// BEFORE: Legacy Go 1.17 style using interface{}
// ============================================================================
package repository

import (
	"context"
	"fmt"
)

type MetadataStore struct {
	data map[string]interface{}
}

func NewMetadataStore() *MetadataStore {
	return &MetadataStore{
		data: make(map[string]interface{}),
	}
}

func (m *MetadataStore) Put(key string, val interface{}) {
	m.data[key] = val
}

func (m *MetadataStore) Get(key string) (interface{}, bool) {
	val, ok := m.data[key]
	return val, ok
}

func FormatItem(item interface{}) string {
	switch v := item.(type) {
	case string:
		return v
	case fmt.Stringer:
		return v.String()
	default:
		return fmt.Sprintf("%v", item)
	}
}

// ============================================================================
// AFTER: Modern Go 1.18+ using any
// ============================================================================
package repository

import (
	"context"
	"fmt"
)

type MetadataStore struct {
	data map[string]any
}

func NewMetadataStore() *MetadataStore {
	return &MetadataStore{
		data: make(map[string]any),
	}
}

func (m *MetadataStore) Put(key string, val any) {
	m.data[key] = val
}

func (m *MetadataStore) Get(key string) (any, bool) {
	val, ok := m.data[key]
	return val, ok
}

func FormatItem(item any) string {
	switch v := item.(type) {
	case string:
		return v
	case fmt.Stringer:
		return v.String()
	default:
		return fmt.Sprintf("%v", item)
	}
}
```

---

### Directive 1.2: Standard Slice Operations (`slices_package`)
- **Version Gate**: Go 1.21+
- **Rationale**: Standard library algorithms in `slices` are highly optimized by the Go compiler, avoid bounds-checking overhead, and eliminate repetitive, error-prone manual loop boilerplate.
- **Model Rule**: Replace manual search loops, element indexing, equality checks, and cloning with `slices` functions.

#### Code Transformation Sample
```go
// ============================================================================
// BEFORE: Manual loops for slice search, copy, and equality
// ============================================================================
package permissions

type Role struct {
	Name        string
	Permissions []string
}

// Manual element search
func HasPermission(roles []Role, roleName, perm string) bool {
	for _, r := range roles {
		if r.Name == roleName {
			for _, p := range r.Permissions {
				if p == perm {
					return true
				}
			}
		}
	}
	return false
}

// Manual slice copy
func CloneRoles(src []Role) []Role {
	if src == nil {
		return nil
	}
	dst := make([]Role, len(src))
	for i, r := range src {
		dst[i] = r
	}
	return dst
}

// Manual slice equality
func EqualPermissions(a, b []string) bool {
	if len(a) != len(b) {
		return false
	}
	for i := range a {
		if a[i] != b[i] {
			return false
		}
	}
	return true
}

// ============================================================================
// AFTER: Modern Go 1.21+ slices standard package
// ============================================================================
package permissions

import "slices"

type Role struct {
	Name        string
	Permissions []string
}

// Idiomatic search using slices.IndexFunc and slices.Contains
func HasPermission(roles []Role, roleName, perm string) bool {
	idx := slices.IndexFunc(roles, func(r Role) bool {
		return r.Name == roleName
	})
	if idx == -1 {
		return false
	}
	return slices.Contains(roles[idx].Permissions, perm)
}

// Idiomatic shallow clone with single allocation
func CloneRoles(src []Role) []Role {
	return slices.Clone(src)
}

// Idiomatic slice comparison
func EqualPermissions(a, b []string) bool {
	return slices.Equal(a, b)
}
```

---

### Directive 1.3: Slice Concatenation & Batching (`slices_concat`, `slices_chunk`)
- **Version Gate**:
  - `slices.Concat`: Go 1.22+
  - `slices.Chunk`: Go 1.23+
- **Rationale**:
  - Chained `append` calls cause multiple slice reallocation steps if capacities exceed. `slices.Concat` pre-calculates the exact total capacity and performs a single allocation.
  - Manual index math for batching `s[i:min(i+batchSize, len(s))]` frequently suffers from off-by-one errors. `slices.Chunk` provides an iterator that cleanly yields batches.

#### Code Transformation Sample
```go
// ============================================================================
// BEFORE: Manual append chains and brittle batch slicing
// ============================================================================
package batcher

import "context"

func CombineFilters(base, tenant, overrides []string) []string {
	var combined []string
	combined = append(combined, base...)
	combined = append(combined, tenant...)
	combined = append(combined, overrides...)
	return combined
}

func DispatchInBatches(ctx context.Context, records []string, batchSize int, dispatch func([]string) error) error {
	total := len(records)
	for i := 0; i < total; i += batchSize {
		end := i + batchSize
		if end > total {
			end = total
		}
		batch := records[i:end]
		if err := dispatch(batch); err != nil {
			return err
		}
	}
	return nil
}

// ============================================================================
// AFTER: Modern Go 1.22+ / Go 1.23+ slices.Concat and slices.Chunk
// ============================================================================
package batcher

import (
	"context"
	"slices"
)

func CombineFilters(base, tenant, overrides []string) []string {
	// Go 1.22+: Single allocation sizing
	return slices.Concat(base, tenant, overrides)
}

func DispatchInBatches(ctx context.Context, records []string, batchSize int, dispatch func([]string) error) error {
	// Go 1.23+: Clean range-over-func iterator chunking
	for batch := range slices.Chunk(records, batchSize) {
		if err := dispatch(batch); err != nil {
			return err
		}
	}
	return nil
}
```

---

### Directive 1.4: Modern Map Manipulation & Built-in Clear (`maps_package`, `builtin_clear`)
- **Version Gate**: Go 1.21+
- **Rationale**: Built-in `clear` and the `maps` package optimize map mutations, avoid allocating intermediate structures, and cleanly express mutations.

#### Code Transformation Sample
```go
// ============================================================================
// BEFORE: Manual loops for map copy, deletion, and buffer reset
// ============================================================================
package session

type SessionManager struct {
	tokens  map[string]int64
	scratch []byte
}

func (s *SessionManager) CopyTokens() map[string]int64 {
	out := make(map[string]int64, len(s.tokens))
	for k, v := range s.tokens {
		out[k] = v
	}
	return out
}

func (s *SessionManager) EvictExpired(now int64) {
	for k, exp := range s.tokens {
		if exp <= now {
			delete(s.tokens, k)
		}
	}
}

func (s *SessionManager) ResetBuffers() {
	for k := range s.tokens {
		delete(s.tokens, k)
	}
	for i := range s.scratch {
		s.scratch[i] = 0
	}
}

// ============================================================================
// AFTER: Modern Go 1.21+ maps package and built-in clear()
// ============================================================================
package session

import "maps"

type SessionManager struct {
	tokens  map[string]int64
	scratch []byte
}

func (s *SessionManager) CopyTokens() map[string]int64 {
	return maps.Clone(s.tokens)
}

func (s *SessionManager) EvictExpired(now int64) {
	// Safe conditional deletion in place
	maps.DeleteFunc(s.tokens, func(k string, exp int64) bool {
		return exp <= now
	})
}

func (s *SessionManager) ResetBuffers() {
	// clear() empties maps and zeroes slice elements in-place with O(1) allocation
	clear(s.tokens)
	clear(s.scratch)
}
```

---

### Directive 1.5: Built-in Min/Max & Fallback Resolution (`builtin_min_max`, `cmp_or`)
- **Version Gate**: Go 1.21+
- **Rationale**:
  - `min` and `max` are compiler intrinsics operating directly on any ordered type (integers, floats, strings) without casting.
  - `cmp.Or` short-circuits evaluation and returns the first non-zero value, replacing nested fallback logic.

#### Code Transformation Sample
```go
// ============================================================================
// BEFORE: Custom clamp helpers and fallback ladders
// ============================================================================
package config

import (
	"math"
	"os"
)

func ClampInt(val, low, high int) int {
	// math.Min and math.Max require float64 conversion
	fVal := float64(val)
	fLow := float64(low)
	fHigh := float64(high)
	return int(math.Min(math.Max(fVal, fLow), fHigh))
}

func DetermineEndpoint(explicit, configVal string) string {
	if explicit != "" {
		return explicit
	}
	if configVal != "" {
		return configVal
	}
	env := os.Getenv("SERVICE_URL")
	if env != "" {
		return env
	}
	return "https://api.default.internal"
}

// ============================================================================
// AFTER: Modern Go 1.21+ min(), max(), and cmp.Or()
// ============================================================================
package config

import (
	"cmp"
	"os"
)

func ClampInt(val, low, high int) int {
	// Native built-in intrinsics: zero conversion, works on all ordered types
	return min(max(val, low), high)
}

func DetermineEndpoint(explicit, configVal string) string {
	// Returns first non-zero/non-empty string cleanly
	return cmp.Or(explicit, configVal, os.Getenv("SERVICE_URL"), "https://api.default.internal")
}
```

---

### Directive 1.6: Modern Loops & Scoping Semantics (`for_range_int`, `loop_variable_scoping`)
- **Version Gate**: Go 1.22+
- **Rationale**:
  - `for i := range n` replaces the classic 3-clause C-style `for i := 0; i < n; i++`.
  - In Go 1.22+, loop variables are per-iteration scoped. Variable shadowing (`v := v`) is now obsolete and should be deleted.

#### Code Transformation Sample
```go
// ============================================================================
// BEFORE: 3-clause loop and obsolete variable shadowing (Go <= 1.21)
// ============================================================================
package worker

import "sync"

func RunPolls(limit int, poll func(int) bool) bool {
	for i := 0; i < limit; i++ {
		if poll(i) {
			return true
		}
	}
	return false
}

func ProcessConcurrently(items []string) {
	var wg sync.WaitGroup
	for _, item := range items {
		item := item // Required in Go <= 1.21 to prevent closure race condition!
		wg.Add(1)
		go func() {
			defer wg.Done()
			handle(item)
		}()
	}
	wg.Wait()
}

// ============================================================================
// AFTER: Modern Go 1.22+ range over integer & clean loop scoping
// ============================================================================
package worker

import "sync"

func RunPolls(limit int, poll func(int) bool) bool {
	// Range over integer
	for i := range limit {
		if poll(i) {
			return true
		}
	}
	return false
}

func ProcessConcurrently(items []string) {
	var wg sync.WaitGroup
	for _, item := range items {
		// Go 1.22+: No "item := item" required! New memory location allocated each loop.
		wg.Add(1)
		go func() {
			defer wg.Done()
			handle(item)
		}()
	}
	wg.Wait()
}
```

---

### Directive 1.7: Custom Iterators (`iterators`)
- **Version Gate**: Go 1.23+
- **Rationale**: Instead of allocating full slices to iterate over trees, graphs, or database cursors, `iter.Seq[V]` and `iter.Seq2[K, V]` yield items on-demand with zero intermediate memory allocations and early-break support.

#### Code Transformation Sample
```go
// ============================================================================
// BEFORE: Allocates entire slice just to iterate over tree elements
// ============================================================================
package tree

type Node struct {
	Key   string
	Left  *Node
	Right *Node
}

type BST struct {
	Root *Node
}

func (t *BST) InOrderKeys() []string {
	var keys []string
	var walk func(n *Node)
	walk = func(n *Node) {
		if n == nil {
			return
		}
		walk(n.Left)
		keys = append(keys, n.Key)
		walk(n.Right)
	}
	walk(t.Root)
	return keys
}

// ============================================================================
// AFTER: Modern Go 1.23+ push iterator (iter.Seq)
// ============================================================================
package tree

import "iter"

type Node struct {
	Key   string
	Left  *Node
	Right *Node
}

type BST struct {
	Root *Node
}

// Zero allocations: yields items lazily with instant break support
func (t *BST) All() iter.Seq[string] {
	return func(yield func(string) bool) {
		var walk func(n *Node) bool
		walk = func(n *Node) bool {
			if n == nil {
				return true
			}
			if !walk(n.Left) {
				return false
			}
			if !yield(n.Key) {
				return false
			}
			return walk(n.Right)
		}
		walk(t.Root)
	}
}

// Usage in callers:
// for key := range bst.All() {
//     if key == "target" { break } // Terminates walk immediately!
// }
```

---

### Directive 1.8: Literal Pointers (`new_expression`)
- **Version Gate**: Go 1.26+
- **Rationale**: `new(value)` allows passing expressions and primitives directly into pointer-accepting struct fields, eliminating generic `ptr()` helpers.

#### Code Transformation Sample
```go
// ============================================================================
// BEFORE: Custom generic pointer helper function
// ============================================================================
package sdk

import "time"

func Ptr[T any](v T) *T {
	return &v
}

type ClientOptions struct {
	Timeout   *time.Duration
	Retries   *int
	DebugMode *bool
}

func DefaultOptions() ClientOptions {
	return ClientOptions{
		Timeout:   Ptr(10 * time.Second),
		Retries:   Ptr(3),
		DebugMode: Ptr(false),
	}
}

// ============================================================================
// AFTER: Modern Go 1.26+ new(expression)
// ============================================================================
package sdk

import "time"

type ClientOptions struct {
	Timeout   *time.Duration
	Retries   *int
	DebugMode *bool
}

func DefaultOptions() ClientOptions {
	return ClientOptions{
		Timeout:   new(10 * time.Second),
		Retries:   new(3),
		DebugMode: new(false),
	}
}
```

---

### Directive 1.9: Next-Gen JSON Serialization (`json_v2`)
- **Version Gate**: Go 1.27+
- **Rationale**: `encoding/json/v2` provides standard omitzero behavior for zero structs and times, eliminating pointer wrapping hacks.

#### Code Transformation Sample
```go
// ============================================================================
// BEFORE: Legacy JSON v1 with pointer hacks for omit empty time.Time
// ============================================================================
package api

import (
	"encoding/json"
	"time"
)

type AuditLog struct {
	Action    string     `json:"action"`
	CreatedAt time.Time  `json:"created_at"`
	// Pointer required because time.Time zero value is not empty struct in v1
	DeletedAt *time.Time `json:"deleted_at,omitempty"`
}

func EncodeLog(l AuditLog) ([]byte, error) {
	return json.Marshal(l)
}

// ============================================================================
// AFTER: Modern Go 1.27+ encoding/json/v2 with omitzero
// ============================================================================
package api

import (
	"encoding/json/v2"
	"time"
)

type AuditLog struct {
	Action    string    `json:"action"`
	CreatedAt time.Time `json:"created_at"`
	// omitzero inspects IsZero() and zero primitives cleanly without pointer overhead
	DeletedAt time.Time `json:"deleted_at,omitzero"`
}

func EncodeLog(l AuditLog) ([]byte, error) {
	return json.Marshal(l)
}
```

---

## 5. Agent Verification Checklist & Handover

Execute the following automated verification suite before handing over:

```bash
# 1. Format all code
go fmt ./...

# 2. Verify compilation
go build ./...

# 3. Static analysis checks
go vet ./...
```

### Handover Artifact
Create `handover-phase-1-report.md` detailing:
- Target Go version verified.
- Files refactored with specific rule IDs.
- Confirmation that `go vet ./...` and `go build ./...` passed.

Hand off execution to **Agent 2** via `actions/02-go-concurrency-systems.action.md`.
