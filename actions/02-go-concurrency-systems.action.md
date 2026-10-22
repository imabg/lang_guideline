---
name: "action-02-go-concurrency-systems"
title: "Modern Go Concurrency, Systems & Networking Architecture Modernization"
version: "1.1.0"
agent_role: "Modern Go Concurrency & Systems Architecture Specialist Agent"
target_scope: "Goroutine lifecycles, synchronization, typed atomics, contexts, HTTP routing, and sandboxed file I/O"
target_languages: ["go"]
phase: 2
handover_input: "Phase 1 verified branch (agent/phase-1-core-data) with modernized data models"
handover_output: "Race-free concurrent systems, modernized HTTP multiplexer, sandboxed I/O, passing go test -race"
next_action: "actions/03-go-errors-testing-observability.action.md"
---

# Action 02: Modern Go Concurrency, Systems & Networking Architecture

## 1. Executive Mission & Agent Role

You are **Agent 2: Concurrency & Systems Specialist**.
Your mission is to execute Phase 2 of building or modernizing a Go repository. You receive a codebase whose data structures, collections, and built-ins have been verified by Agent 1. You modernize goroutine orchestration, memory synchronization, context cancellation lifecycles, atomic operations, HTTP multiplexing, and secure filesystem access.

### Scope Boundaries
- **IN SCOPE**:
  - Goroutine spawning and group synchronization (`sync.WaitGroup.Go`).
  - Thread-safe memory primitives (`sync/atomic` typed structs: `atomic.Int64`, `atomic.Uint64`, `atomic.Bool`, `atomic.Pointer[T]`).
  - Context cancellation cause tracking (`context.WithCancelCause`, `context.Cause`, and `context.WithoutCancel`).
  - HTTP request multiplexing and path routing (`http.ServeMux` with method matching and `{param}` wildcards).
  - Directory-scoped sandboxed filesystem access (`os.OpenRoot`, `os.Root`).
  - Zero-allocation network IP manipulation (`netip.Addr`, `netip.Prefix`).
  - Goroutine leak prevention, channel draining, and shutdown coordination.
- **OUT OF SCOPE** (Addressed by other agents):
  - Collections, slices, maps, and built-ins (*Completed by Agent 1*).
  - Benchmarks (`b.Loop()`), test contexts (`t.Context()`), synctest, and structured logging (*Handed to Agent 3*).

---

## 2. Model Decision Heuristics & Decision Matrices

To ensure AI models choose the correct concurrency and systems architecture, apply these decision rules:

### Decision Matrix 1: Goroutine Orchestration & Synchronization
When an agent encounters concurrent workloads:

| Concurrency Scenario | Recommended Construct | Why / Decision Rationale | Common Model Anti-Pattern |
| :--- | :--- | :--- | :--- |
| **Independent background workers without caller error return** | `sync.WaitGroup.Go(fn)` (Go 1.24+) | Automatically manages counter increments and decrements, eliminating panic-prone `wg.Add(1)` mismatches. | Raw `go func() { defer wg.Done() }` without pairing `wg.Add(1)`. |
| **Concurrent tasks where ANY error should abort remaining tasks** | `golang.org/x/sync/errgroup` with `Group.WithContext(ctx)` | Propagates context cancellation upon the first worker error; returns the first non-nil error. | Manually collecting errors in slices with mutexes and manually canceling contexts. |
| **Rate-limited worker pool (fixed concurrency limit)** | Buffered channel semaphore (`sem := make(chan struct{}, limit)`) | Bound memory usage; prevents spawning $100,000$ simultaneous goroutines under load spikes. | Spawning unbounded goroutines inside HTTP handlers or file loops. |

### Decision Matrix 2: Memory Synchronization & State Sharing
When multiple goroutines access shared state:

| Workload Access Profile | Recommended Construct | Performance / Complexity Profile | Key Rule |
| :--- | :--- | :--- | :--- |
| **Read-heavy configuration / routing table (99%+ reads)** | `atomic.Pointer[T]` (Go 1.18+) | Zero lock contention on reads; atomic pointer swap on write (Copy-On-Write). | Treat the dereferenced struct as completely immutable. |
| **Simple numeric metrics / counters / flags** | `atomic.Int64`, `atomic.Uint64`, `atomic.Bool` (Go 1.18+) | Hardware CPU atomic instruction ($O(1)$ CPU cycle, no OS mutex sleep). | Do NOT mix with mutexes. Never take address `&atomic` for legacy calls. |
| **Complex multi-field state invariants** | `sync.RWMutex` or `sync.Mutex` | Coordinates multi-variable updates atomically. | Keep critical sections minimal; never perform I/O or network calls holding a mutex! |
| **Producer-Consumer data handoff** | Go Channels (`chan T`) | Communicates by sharing memory; clear single ownership. | Always close channels from the PRODUCER side only; never from the consumer. |

### Decision Matrix 3: Context Propagation & Root Cause Isolation
| Context Need | Recommended Pattern | When NOT to use |
| :--- | :--- | :--- |
| **Cancellation triggered by application error** | `context.WithCancelCause` (Go 1.20+) | Do not use if cancellation is purely time-based (use `context.WithTimeout`). |
| **Detaching background tasks from HTTP abort** | `context.WithoutCancel(ctx)` (Go 1.21+) | Do not use if the background task truly SHOULD stop when the client disconnects. |

### Decision Matrix 4: Sandboxed Filesystem Access
| Security Requirement | Recommended Construct | Danger of Legacy Alternatives |
| :--- | :--- | :--- |
| **Opening user-supplied relative paths within a root directory** | `os.OpenRoot(baseDir)` (Go 1.24+) | `filepath.Clean` and `filepath.Join` do NOT protect against symlink directory escapes! |

---

## 3. Version Compatibility Matrix

| Rule ID | Minimum Go | Modern Construct | Legacy Construct |
| :--- | :--- | :--- | :--- |
| `netip_addr` | Go 1.18+ | `netip.Addr`, `netip.Prefix` | `net.IP`, `net.IPNet` |
| `atomic_typed` | Go 1.18+ | `atomic.Int64`, `atomic.Bool`, `atomic.Pointer[T]` | `atomic.AddInt64(&v, 1)`, `unsafe.Pointer` |
| `context_with_cancel_cause` | Go 1.20+ | `context.WithCancelCause`, `context.Cause` | `context.WithCancel` |
| `context_without_cancel` | Go 1.21+ | `context.WithoutCancel(ctx)` | Re-creating empty `context.Background()` |
| `http_routing_patterns` | Go 1.22+ | `http.NewServeMux()` with `"GET /api/{id}"` | `chi.Router`, `gorilla/mux`, manual path switch |
| `sync_waitgroup_go` | Go 1.24+ | `wg.Go(fn)` | `wg.Add(1)` + `go func() { defer wg.Done() }` |
| `os_root` | Go 1.24+ | `os.OpenRoot(dir)`, `root.Open(rel)` | `filepath.Join(dir, rel)` + `filepath.Clean` |

---

## 4. Detailed Directives with Production-Grade Code Samples

---

### Directive 2.1: Modern Atomic Memory Primitives (`atomic_typed`)
- **Version Gate**: Go 1.18+
- **Rationale**: Typed atomics eliminate raw memory pointer arithmetic, provide compiler type safety, prevent incorrect size operations (such as calling 32-bit atomic operations on 64-bit fields), and read much cleaner.
- **Model Rule**: Replace `sync/atomic` functions taking pointer addresses (`atomic.LoadInt64(&v)`) with method invocations on typed structs (`v.Load()`).

#### Code Transformation Sample
```go
// ============================================================================
// BEFORE: Legacy untyped pointer atomics (Go <= 1.17)
// ============================================================================
package gateway

import (
	"sync/atomic"
	"unsafe"
)

type RouteConfig struct {
	BackendURL string
	MaxConns   int
}

type GatewayMetrics struct {
	requestCount int64
	inMaintenance int32 // 0 = false, 1 = true
	activeConfig  unsafe.Pointer // *RouteConfig
}

func (g *GatewayMetrics) IncRequests() {
	atomic.AddInt64(&g.requestCount, 1)
}

func (g *GatewayMetrics) GetRequests() int64 {
	return atomic.LoadInt64(&g.requestCount)
}

func (g *GatewayMetrics) SetMaintenance(enabled bool) {
	var val int32
	if enabled {
		val = 1
	}
	atomic.StoreInt32(&g.inMaintenance, val)
}

func (g *GatewayMetrics) SwapConfig(cfg *RouteConfig) {
	atomic.StorePointer(&g.activeConfig, unsafe.Pointer(cfg))
}

func (g *GatewayMetrics) CurrentConfig() *RouteConfig {
	ptr := atomic.LoadPointer(&g.activeConfig)
	if ptr == nil {
		return nil
	}
	return (*RouteConfig)(ptr)
}

// ============================================================================
// AFTER: Modern Go 1.18+ typed atomics
// ============================================================================
package gateway

import "sync/atomic"

type RouteConfig struct {
	BackendURL string
	MaxConns   int
}

type GatewayMetrics struct {
	requestCount  atomic.Int64
	inMaintenance atomic.Bool
	activeConfig  atomic.Pointer[RouteConfig]
}

func (g *GatewayMetrics) IncRequests() {
	g.requestCount.Add(1)
}

func (g *GatewayMetrics) GetRequests() int64 {
	return g.requestCount.Load()
}

func (g *GatewayMetrics) SetMaintenance(enabled bool) {
	g.inMaintenance.Store(enabled)
}

func (g *GatewayMetrics) SwapConfig(cfg *RouteConfig) {
	g.activeConfig.Store(cfg)
}

func (g *GatewayMetrics) CurrentConfig() *RouteConfig {
	return g.activeConfig.Load()
}
```

---

### Directive 2.2: Automated WaitGroup Goroutines (`sync_waitgroup_go`)
- **Version Gate**: Go 1.24+
- **Rationale**: `wg.Go(fn)` binds goroutine creation directly to the WaitGroup counter. It guarantees that `Add(1)` is executed prior to goroutine scheduling and that `Done()` is always called on function exit, even if the function panics.
- **Model Rule**: Replace `wg.Add(1)` + `go func() { defer wg.Done(); ... }()` with `wg.Go(...)`.

#### Code Transformation Sample
```go
// ============================================================================
// BEFORE: Manual WaitGroup counters with error-prone placement
// ============================================================================
package crawler

import (
	"context"
	"sync"
)

type Fetcher struct {
	urls []string
}

func (f *Fetcher) FetchAll(ctx context.Context, handler func(string)) {
	var wg sync.WaitGroup

	for _, url := range f.urls {
		wg.Add(1)
		go func(u string) {
			defer wg.Done()
			select {
			case <-ctx.Done():
				return
			default:
				handler(u)
			}
		}(url)
	}

	wg.Wait()
}

// ============================================================================
// AFTER: Modern Go 1.24+ sync.WaitGroup.Go
// ============================================================================
package crawler

import (
	"context"
	"sync"
)

type Fetcher struct {
	urls []string
}

func (f *Fetcher) FetchAll(ctx context.Context, handler func(string)) {
	var wg sync.WaitGroup

	for _, url := range f.urls {
		// Go 1.24+: Automatically increments counter and ensures Done() on exit
		wg.Go(func() {
			select {
			case <-ctx.Done():
				return
			default:
				handler(url)
			}
		})
	}

	wg.Wait()
}
```

---

### Directive 2.3: Context Root Cause Propagation (`context_with_cancel_cause`)
- **Version Gate**: Go 1.20+
- **Rationale**: Standard `context.WithCancel` discards the underlying error that triggered cancellation, leaving callers with generic `context.Canceled`. `context.WithCancelCause` allows associating a specific error with cancellation, inspectable via `context.Cause(ctx)`.

#### Code Transformation Sample
```go
// ============================================================================
// BEFORE: Context cancellation loses the operational failure reason
// ============================================================================
package consumer

import (
	"context"
	"errors"
	"fmt"
)

var ErrHeartbeatLost = errors.New("heartbeat lost to broker")

type StreamConsumer struct{}

func (s *StreamConsumer) Consume(parentCtx context.Context) error {
	ctx, cancel := context.WithCancel(parentCtx)
	defer cancel()

	go func() {
		if err := s.monitorHeartbeat(); err != nil {
			// Cancels context, but ErrHeartbeatLost is completely lost to the caller!
			cancel()
		}
	}()

	<-ctx.Done()
	// ctx.Err() only returns context.Canceled!
	return ctx.Err()
}

func (s *StreamConsumer) monitorHeartbeat() error {
	return ErrHeartbeatLost
}

// ============================================================================
// AFTER: Modern Go 1.20+ context.WithCancelCause and context.Cause
// ============================================================================
package consumer

import (
	"context"
	"errors"
	"fmt"
)

var ErrHeartbeatLost = errors.New("heartbeat lost to broker")

type StreamConsumer struct{}

func (s *StreamConsumer) Consume(parentCtx context.Context) error {
	ctx, cancel := context.WithCancelCause(parentCtx)
	defer cancel(nil)

	go func() {
		if err := s.monitorHeartbeat(); err != nil {
			// Propagate the specific root cause error
			cancel(fmt.Errorf("consumer aborted: %w", err))
		}
	}()

	<-ctx.Done()
	// Inspect root cause
	if cause := context.Cause(ctx); cause != nil && !errors.Is(cause, context.Canceled) {
		return cause
	}
	return ctx.Err()
}

func (s *StreamConsumer) monitorHeartbeat() error {
	return ErrHeartbeatLost
}
```

---

### Directive 2.4: Standard Library HTTP Method & Wildcard Routing (`http_routing_patterns`)
- **Version Gate**: Go 1.22+
- **Rationale**: Modern `http.ServeMux` natively handles HTTP method dispatching (`GET`, `POST`, `DELETE`) and wildcard path matching (`/users/{id}`, `/files/{path...}`). External routing dependencies like Chi or Gorilla Mux are no longer necessary for REST endpoints.
- **Model Rule**: Replace external routers or manual string splitting with `http.NewServeMux` and `r.PathValue("key")`.

#### Code Transformation Sample
```go
// ============================================================================
// BEFORE: Manual path segment splitting and method switches
// ============================================================================
package server

import (
	"fmt"
	"net/http"
	"strings"
)

type UserAPI struct{}

func (u *UserAPI) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	// Manual URL path splitting
	trimmed := strings.Trim(r.URL.Path, "/")
	parts := strings.Split(trimmed, "/")

	if len(parts) == 2 && parts[0] == "users" {
		userID := parts[1]
		switch r.Method {
		case http.MethodGet:
			fmt.Fprintf(w, "Fetched user %s", userID)
			return
		case http.MethodDelete:
			fmt.Fprintf(w, "Deleted user %s", userID)
			return
		default:
			http.Error(w, "Method Not Allowed", http.StatusMethodNotAllowed)
			return
		}
	}

	http.NotFound(w, r)
}

// ============================================================================
// AFTER: Modern Go 1.22+ Enhanced http.ServeMux
// ============================================================================
package server

import (
	"fmt"
	"net/http"
)

func NewRouter() http.Handler {
	mux := http.NewServeMux()

	// Method-prefixed pattern with path parameters
	mux.HandleFunc("GET /users/{id}", func(w http.ResponseWriter, r *http.Request) {
		userID := r.PathValue("id")
		fmt.Fprintf(w, "Fetched user %s", userID)
	})

	mux.HandleFunc("DELETE /users/{id}", func(w http.ResponseWriter, r *http.Request) {
		userID := r.PathValue("id")
		fmt.Fprintf(w, "Deleted user %s", userID)
	})

	// Subtree catch-all wildcard
	mux.HandleFunc("GET /assets/{filepath...}", func(w http.ResponseWriter, r *http.Request) {
		path := r.PathValue("filepath")
		fmt.Fprintf(w, "Serving asset %s", path)
	})

	return mux
}
```

---

### Directive 2.5: Sandboxed Filesystem Access (`os_root`)
- **Version Gate**: Go 1.24+
- **Rationale**: User-supplied file paths are a frequent source of Local File Inclusion (LFI) vulnerabilities. `filepath.Clean` and string prefix checks can be defeated by symlinks. `os.OpenRoot` restricts file access to the specified directory using OS-level directory file descriptors.
- **Model Rule**: Replace `filepath.Join(base, untrusted)` with `os.OpenRoot(base)`.

#### Code Transformation Sample
```go
// ============================================================================
// BEFORE: Fragile filepath checks vulnerable to symlink traversal
// ============================================================================
package docstore

import (
	"errors"
	"os"
	"path/filepath"
	"strings"
)

func ReadDocument(storageDir, relativePath string) ([]byte, error) {
	target := filepath.Join(storageDir, relativePath)
	cleaned := filepath.Clean(target)

	// Insecure: If storageDir contains a symlink pointing to /etc, check passes!
	if !strings.HasPrefix(cleaned, filepath.Clean(storageDir)) {
		return nil, errors.New("forbidden: path traversal attempt")
	}

	return os.ReadFile(cleaned)
}

// ============================================================================
// AFTER: Modern Go 1.24+ os.OpenRoot Jailed Filesystem
// ============================================================================
package docstore

import (
	"io"
	"os"
)

func ReadDocument(storageDir, relativePath string) ([]byte, error) {
	// Opens a restricted root jail
	root, err := os.OpenRoot(storageDir)
	if err != nil {
		return nil, err
	}
	defer root.Close()

	// Kernel enforces that relativePath cannot escape storageDir under any circumstances
	f, err := root.Open(relativePath)
	if err != nil {
		return nil, err
	}
	defer f.Close()

	return io.ReadAll(f)
}
```

---

### Directive 2.6: Zero-Allocation Network Addressing (`netip_addr`)
- **Version Gate**: Go 1.18+
- **Rationale**: `net.IP` is a slice of bytes (`[]byte`). It allocates memory on the heap, is mutable, and cannot be used as a map key. `netip.Addr` is a 24-byte immutable value type that requires zero heap allocations and is natively comparable.
- **Model Rule**: Replace `net.IP` and `net.IPNet` with `netip.Addr` and `netip.Prefix`.

#### Code Transformation Sample
```go
// ==========================================
// BEFORE: Legacy net.IP with heap allocations
// ==========================================
package firewall

import (
	"net"
)

type IPFilter struct {
	// Cannot use map[net.IP]bool because slices are not comparable!
	blockList []net.IP
}

func (f *IPFilter) Block(ipStr string) {
	ip := net.ParseIP(ipStr)
	if ip != nil {
		f.blockList = append(f.blockList, ip)
	}
}

func (f *IPFilter) IsBlocked(ipStr string) bool {
	parsed := net.ParseIP(ipStr)
	if parsed == nil {
		return false
	}
	for _, blocked := range f.blockList {
		if blocked.Equal(parsed) {
			return true
		}
	}
	return false
}

// ==========================================
// AFTER: Modern Go 1.18+ netip.Addr
// ==========================================
package firewall

import "net/netip"

type IPFilter struct {
	// netip.Addr is comparable; supports O(1) map lookups with zero heap allocations
	blockMap map[netip.Addr]struct{}
}

func NewIPFilter() *IPFilter {
	return &IPFilter{blockMap: make(map[netip.Addr]struct{})}
}

func (f *IPFilter) Block(ipStr string) error {
	addr, err := netip.ParseAddr(ipStr)
	if err != nil {
		return err
	}
	f.blockMap[addr] = struct{}{}
	return nil
}

func (f *IPFilter) IsBlocked(ipStr string) bool {
	addr, err := netip.ParseAddr(ipStr)
	if err != nil {
		return false
	}
	_, found := f.blockMap[addr]
	return found
}
```

---

## 5. Agent Verification Checklist & Handover

Execute the following verification sequence:

```bash
# 1. Format code
go fmt ./...

# 2. Compile all packages
go build ./...

# 3. Race condition verification
go test -race -run=^$ ./...

# 4. Static checks
go vet ./...
```

### Handover Artifact
Create `handover-phase-2-report.md` detailing:
- Concurrency systems modernized.
- Race detector verification results.
- Transition notes for **Agent 3: Error Handling, Testing & Observability**.

Hand off execution to **Agent 3** via `actions/03-go-errors-testing-observability.action.md`.
