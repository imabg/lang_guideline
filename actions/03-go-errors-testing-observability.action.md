---
name: "action-03-go-errors-testing-observability"
title: "Modern Go Error Handling, Testing & Observability Modernization"
version: "1.1.0"
agent_role: "Modern Go Error Handling, Testing & Observability Specialist Agent"
target_scope: "Error wrapping, multi-errors, type-safe error matching, benchmark loops, test contexts, synctest, and structured slog"
target_languages: ["go"]
phase: 3
handover_input: "Phase 2 verified branch (agent/phase-2-concurrency-systems) with clean concurrency models"
handover_output: "Fully modernized, resilient, observed, and verified codebase ready for production merge"
next_action: "DONE (Creates final consolidated PR report)"
---

# Action 03: Modern Go Error Handling, Testing & Observability

## 1. Executive Mission & Agent Role

You are **Agent 3: Error Handling, Testing & Observability Specialist**.
Your mission is to execute Phase 3 (the final phase) of modernizing a Go codebase. You receive a codebase whose core collections and concurrency architectures have already been modernized by Agent 1 and Agent 2. You complete the transformation by enforcing left-aligned error resilience, modernizing unit test harnesses, upgrading benchmark loops, incorporating synthetic time testing, and upgrading legacy logging to structured `log/slog`.

### Scope Boundaries
- **IN SCOPE**:
  - Control flow: Enforce left-aligned happy path (guard clauses, early returns).
  - Error wrapping: Contextual wrapping using `fmt.Errorf("...: %w", err)`.
  - Multi-error aggregation: `errors.Join(err1, err2, ...)`.
  - Type-safe error unwrapping: `errors.AsType[T](err)`.
  - Modern benchmark loops: `for b.Loop()`.
  - Scoped test context management: `t.Context()`.
  - Deterministic concurrent time testing: `testing/synctest` (`synctest.Run`, `synctest.Wait`).
  - Structured, contextual key-value logging: `log/slog` with `slog.InfoContext` and typed attributes.
- **OUT OF SCOPE** (Addressed in earlier phases):
  - Slice/map algorithms and built-ins (*Completed by Agent 1*).
  - Goroutines, atomics, and HTTP mux routing (*Completed by Agent 2*).

---

## 2. Model Decision Heuristics & Decision Matrices

To ensure AI models take the correct engineering decisions in error handling, testing, and observability:

### Decision Matrix 1: Choosing Error Handling Pattern
When an error condition is encountered:

| Scenario / Need | Recommended Construct | Why / Decision Rationale | Anti-Pattern to Avoid |
| :--- | :--- | :--- | :--- |
| **Enriching error with local context** | `fmt.Errorf("parsing user %s: %w", id, err)` | Adds human-readable context while preserving the error chain for `errors.Is`/`errors.As`. | Discarding original error with `fmt.Errorf("%v", err)` (breaks wrapping!). |
| **Aggregating multiple errors (e.g. multi-resource cleanup, form validation)** | `errors.Join(errs...)` (Go 1.20+) | Combines errors into a single error tree; preserves unwrap capabilities for all items. | Creating custom `[]error` structs or dropping deferred cleanup errors. |
| **Comparing against sentinel error value** | `errors.Is(err, ErrNotFound)` | Traverses the entire error chain to find a matching sentinel. | `if err == ErrNotFound` (fails if error was wrapped!). |
| **Extracting error payload / struct fields** | `errors.AsType[T](err)` (Go 1.26+) or `errors.As(err, &target)` | Extracts the specific type safely without panic. | Direct type assertion `err.(*CustomError)` (panics if wrapped!). |
| **Control Flow & Guard Clauses** | Early return on error condition | Keeps happy path left-aligned at column 0. | Nested `if err == nil { ... } else { return err }` pyramids. |

### Decision Matrix 2: Choosing Test & Benchmark Idioms
| Scenario / Need | Recommended Modern Pattern | Why / Decision Rationale |
| :--- | :--- | :--- |
| **Benchmark execution loop** | `for b.Loop()` (Go 1.24+) | Isolates setup code, eliminates manual `b.ResetTimer()`, prevents loop variable math bugs. |
| **Context needed for test/subtest** | `t.Context()` (Go 1.24+) | Automatically canceled on test teardown; prevents goroutine leaks across subtests. |
| **Testing time-dependent concurrent code (timers, tickers, debouncers)** | `synctest.Run` (Go 1.24+) | Runs in virtual synthetic time bubble; instantaneous execution without flaky `time.Sleep`. |

### Decision Matrix 3: Structured Logging Strategy
| Context | Recommended Modern Pattern | Why / Decision Rationale |
| :--- | :--- | :--- |
| **Service / HTTP request handlers** | `slog.InfoContext(ctx, "msg", attrs...)` | Propagates trace/span IDs and request context into logs. |
| **High-throughput hot paths** | Typed attributes: `slog.Int("count", n)`, `slog.Duration("took", d)` | Zero-allocation; avoids interface boxing overhead. |
| **Application-wide output format** | `slog.NewJSONHandler` for production; `slog.NewTextHandler` for local CLI | Consistent machine-readable JSON in production environments. |

---

## 3. Version Compatibility Matrix

| Rule ID | Minimum Go | Modern Construct | Legacy Construct |
| :--- | :--- | :--- | :--- |
| `happy_path` | Any Go | Guard clauses with early returns | Nested `if-else` cascades |
| `errors_join` | Go 1.20+ | `errors.Join(err1, err2)` | Custom multierror structs, discarded errors |
| `log_slog` | Go 1.21+ | `log/slog` | `log.Printf`, `fmt.Println`, external loggers |
| `testing_b_loop` | Go 1.24+ | `for b.Loop()` | `b.ResetTimer()` + `for i := 0; i < b.N; i++` |
| `testing_t_context` | Go 1.24+ | `t.Context()` | `context.WithCancel(context.Background())` |
| `synctest` | Go 1.24+ | `testing/synctest.Run` | Flaky `time.Sleep(...)` in tests |
| `errors_as_type` | Go 1.26+ | `errors.AsType[T](err)` | `var target T; errors.As(err, &target)` |

---

## 4. Detailed Directives with Production-Grade Code Samples

---

### Directive 3.1: Happy Path Left-Aligned Control Flow (`happy_path`)
- **Version Gate**: All Go versions.
- **Rationale**: Deeply nested `if-else` cascades obscure the main execution path and increase cognitive load. Left-aligned code handles errors immediately with early returns, ensuring the happy path stays unindented.

#### Code Transformation Sample
```go
// ============================================================================
// BEFORE: Deeply nested pyramid indentation (Arrow Anti-Pattern)
// ============================================================================
package checkout

import (
	"errors"
	"fmt"
)

type OrderService struct{}

func (s *OrderService) CompleteCheckout(order *Order) (*Receipt, error) {
	if order != nil {
		if len(order.Items) > 0 {
			if order.Customer.Email != "" {
				res, err := chargeGateway(order)
				if err == nil {
					receipt, err := recordOrder(order, res)
					if err == nil {
						return receipt, nil
					} else {
						return nil, fmt.Errorf("record order error: %w", err)
					}
				} else {
					return nil, fmt.Errorf("charge error: %w", err)
				}
			} else {
				return nil, errors.New("missing customer email")
			}
		} else {
			return nil, errors.New("order has no items")
		}
	} else {
		return nil, errors.New("order cannot be nil")
	}
}

// ============================================================================
// AFTER: Modern Go Left-Aligned Happy Path
// ============================================================================
package checkout

import (
	"errors"
	"fmt"
)

type OrderService struct{}

func (s *OrderService) CompleteCheckout(order *Order) (*Receipt, error) {
	if order == nil {
		return nil, errors.New("order cannot be nil")
	}
	if len(order.Items) == 0 {
		return nil, errors.New("order has no items")
	}
	if order.Customer.Email == "" {
		return nil, errors.New("missing customer email")
	}

	res, err := chargeGateway(order)
	if err != nil {
		return nil, fmt.Errorf("charge error: %w", err)
	}

	receipt, err := recordOrder(order, res)
	if err != nil {
		return nil, fmt.Errorf("record order error: %w", err)
	}

	return receipt, nil
}
```

---

### Directive 3.2: Multi-Error Aggregation (`errors_join`)
- **Version Gate**: Go 1.20+
- **Rationale**: `errors.Join` aggregates multiple errors into a single error while preserving the ability for `errors.Is` and `errors.As` to match any individual component error. It eliminates third-party multierror libraries.

#### Code Transformation Sample
```go
// ============================================================================
// BEFORE: Manual error slice or discarded cleanup error
// ============================================================================
package storage

import (
	"fmt"
	"io"
	"os"
)

type Transaction struct {
	f1 *os.File
	f2 *os.File
}

func (t *Transaction) Close() error {
	var err1, err2 error
	if t.f1 != nil {
		err1 = t.f1.Close()
	}
	if t.f2 != nil {
		err2 = t.f2.Close()
	}

	// Incomplete error reporting: if both fail, one error is dropped or formatted manually
	if err1 != nil && err2 != nil {
		return fmt.Errorf("f1: %v; f2: %v", err1, err2)
	}
	if err1 != nil {
		return err1
	}
	return err2
}

// ============================================================================
// AFTER: Modern Go 1.20+ errors.Join
// ============================================================================
package storage

import (
	"errors"
	"os"
)

type Transaction struct {
	f1 *os.File
	f2 *os.File
}

func (t *Transaction) Close() error {
	var errs []error

	if t.f1 != nil {
		if err := t.f1.Close(); err != nil {
			errs = append(errs, err)
		}
	}
	if t.f2 != nil {
		if err := t.f2.Close(); err != nil {
			errs = append(errs, err)
		}
	}

	// Returns nil if errs is empty; combines all errors with full unwrapping support
	return errors.Join(errs...)
}
```

---

### Directive 3.3: Type-Safe Error Extraction (`errors_as_type`)
- **Version Gate**: Go 1.26+
- **Rationale**: `errors.AsType[T](err)` provides a clean generic expression that checks and extracts the error in a single assignment, eliminating the need to declare an uninitialized variable before the check.

#### Code Transformation Sample
```go
// ============================================================================
// BEFORE: Legacy errors.As requiring uninitialized target variable
// ============================================================================
package client

import (
	"errors"
	"io/fs"
	"log"
)

func InspectPathError(err error) {
	if err == nil {
		return
	}

	// Legacy syntax: must declare uninitialized variable and pass pointer to pointer
	var pathErr *fs.PathError
	if errors.As(err, &pathErr) {
		log.Printf("failed accessing file at %s: %v", pathErr.Path, pathErr.Err)
		return
	}

	log.Printf("generic error: %v", err)
}

// ============================================================================
// AFTER: Modern Go 1.26+ errors.AsType
// ============================================================================
package client

import (
	"errors"
	"io/fs"
	"log/slog"
)

func InspectPathError(err error) {
	if err == nil {
		return
	}

	// Modern Go 1.26+: Direct type extraction and boolean test
	if pathErr, ok := errors.AsType[*fs.PathError](err); ok {
		slog.Error("failed accessing file",
			slog.String("path", pathErr.Path),
			slog.String("op", pathErr.Op),
			slog.Any("err", pathErr.Err),
		)
		return
	}

	slog.Error("generic error occurred", slog.Any("err", err))
}
```

---

### Directive 3.4: Modern Benchmark Loops (`testing_b_loop`)
- **Version Gate**: Go 1.24+
- **Rationale**: `for b.Loop()` cleanly encapsulates benchmark iterations. It isolates setup code so expensive initialization is not measured, eliminates the need for manual `b.ResetTimer()`, and guarantees accurate iteration counting.

#### Code Transformation Sample
```go
// ============================================================================
// BEFORE: Legacy benchmark loop with manual timer resets
// ============================================================================
package crypto_test

import (
	"testing"
	"myproject/crypto"
)

func BenchmarkHashPassword(b *testing.B) {
	// Expensive setup
	salt := crypto.GenerateSalt(32)
	password := "super-secure-master-password"

	// Must remember to reset timer, or setup is included in ns/op!
	b.ResetTimer()

	for i := 0; i < b.N; i++ {
		hash, err := crypto.Hash(password, salt)
		if err != nil {
			b.Fatal(err)
		}
		_ = hash
	}
}

// ============================================================================
// AFTER: Modern Go 1.24+ for b.Loop()
// ============================================================================
package crypto_test

import (
	"testing"
	"myproject/crypto"
)

func BenchmarkHashPassword(b *testing.B) {
	// Setup code outside b.Loop() is automatically excluded from timing
	salt := crypto.GenerateSalt(32)
	password := "super-secure-master-password"

	// b.Loop() automatically handles warmups, measurement, and teardown
	for b.Loop() {
		hash, err := crypto.Hash(password, salt)
		if err != nil {
			b.Fatal(err)
		}
		_ = hash
	}
}
```

---

### Directive 3.5: Scoped Test Contexts (`testing_t_context`)
- **Version Gate**: Go 1.24+
- **Rationale**: Tests frequently create `context.WithCancel(context.Background())` and forget to call `cancel()`, causing goroutines to leak across subtests. `t.Context()` automatically associates the context lifetime with the test or subtest.

#### Code Transformation Sample
```go
// ============================================================================
// BEFORE: Manual context boilerplate prone to goroutine leaks
// ============================================================================
package service_test

import (
	"context"
	"testing"
	"time"
	"myproject/service"
)

func TestServiceProcess(t *testing.T) {
	ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
	defer cancel()

	svc := service.NewService()
	res, err := svc.Execute(ctx, "req-123")
	if err != nil {
		t.Fatalf("unexpected error: %v", err)
	}
	if res.Status != "OK" {
		t.Errorf("expected OK, got: %s", res.Status)
	}
}

// ============================================================================
// AFTER: Modern Go 1.24+ t.Context()
// ============================================================================
package service_test

import (
	"testing"
	"myproject/service"
)

func TestServiceProcess(t *testing.T) {
	svc := service.NewService()

	// Automatically canceled when TestServiceProcess completes
	res, err := svc.Execute(t.Context(), "req-123")
	if err != nil {
		t.Fatalf("unexpected error: %v", err)
	}
	if res.Status != "OK" {
		t.Errorf("expected OK, got: %s", res.Status)
	}
}
```

---

### Directive 3.6: Fast Concurrent Time Testing (`synctest`)
- **Version Gate**: Go 1.24+
- **Rationale**: Tests using real `time.Sleep(...)` are slow and notoriously flaky on shared CI runners. `testing/synctest` creates a synthetic virtual time bubble where time advances instantly whenever all goroutines in the bubble are blocked.

#### Code Transformation Sample
```go
// ============================================================================
// BEFORE: Flaky real-world sleep timers in concurrency tests
// ============================================================================
package ratelimit_test

import (
	"testing"
	"time"
	"myproject/ratelimit"
)

func TestRateLimiterRefill(t *testing.T) {
	rl := ratelimit.NewLimiter(10, 100*time.Millisecond) // 10 tokens per 100ms
	rl.Take(10) // Drain bucket

	// Fragile! Waits 150ms of real clock time. Fails if CI machine is under high load.
	time.Sleep(150 * time.Millisecond)

	if !rl.Allow() {
		t.Fatal("expected tokens to be refilled after 150ms")
	}
}

// ============================================================================
// AFTER: Modern Go 1.24+ testing/synctest Virtual Time Bubble
// ============================================================================
package ratelimit_test

import (
	"testing"
	"testing/synctest"
	"time"
	"myproject/ratelimit"
)

func TestRateLimiterRefill(t *testing.T) {
	synctest.Run(func() {
		rl := ratelimit.NewLimiter(10, 100*time.Millisecond)
		rl.Take(10)

		// Advances synthetic clock instantaneously; waits for background goroutine to catch up
		time.Sleep(150 * time.Millisecond)
		synctest.Wait()

		if !rl.Allow() {
			t.Fatal("expected tokens to be refilled after 150ms")
		}
	})
}
```

---

### Directive 3.7: Structured Logging with Slog (`log_slog`)
- **Version Gate**: Go 1.21+
- **Rationale**: Unstructured logging (`log.Printf`) outputs raw strings that cannot be indexed, queried, or filtered by log aggregators. `log/slog` provides structured key-value logging with contextual trace propagation and zero heap allocations.

#### Code Transformation Sample
```go
// ============================================================================
// BEFORE: Unstructured string formatting with standard log package
// ============================================================================
package server

import (
	"log"
	"net/http"
	"time"
)

func RequestLogger(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		start := time.Now()
		next.ServeHTTP(w, r)
		elapsed := time.Since(start)

		// Unstructured string cannot be queried in Datadog/CloudWatch
		log.Printf("method=%s path=%s remote=%s duration=%v",
			r.Method, r.URL.Path, r.RemoteAddr, elapsed)
	})
}

// ============================================================================
// AFTER: Modern Go 1.21+ log/slog Structured Logging
// ============================================================================
package server

import (
	"log/slog"
	"net/http"
	"time"
)

func RequestLogger(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		start := time.Now()
		next.ServeHTTP(w, r)
		elapsed := time.Since(start)

		// Structured key-value logging with typed attributes and request context
		slog.InfoContext(r.Context(), "http_request_handled",
			slog.String("method", r.Method),
			slog.String("path", r.URL.Path),
			slog.String("remote_addr", r.RemoteAddr),
			slog.Duration("duration", elapsed),
		)
	})
}
```

---

## 5. Final Verification & Modernization Sign-Off Protocol

To certify the repository is fully modernized across all phases:

```bash
# 1. Format entire codebase
go fmt ./...

# 2. Compile all packages, tests, and benchmarks
go test -exec true ./...

# 3. Execute all unit and concurrent tests with race detector
go test -v -race ./...

# 4. Verify benchmarks execute cleanly
go test -run=^$ -bench=. ./...

# 5. Run static analysis
go vet ./...
```

### Final Modernization Sign-Off Report
Create `final-modernization-audit-report.md` consolidating all Phase 1, Phase 2, and Phase 3 improvements, ready for PR merge into `main`.
