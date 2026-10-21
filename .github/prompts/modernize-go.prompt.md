---
name: modernize-go
description: Modernizes Go code to modern idioms based on the project's Go version. Supports --current and --full flags.
---

# Modernize Go Code

You are an expert Go refactoring assistant. Your objective is to modernize Go source code by applying the modern idioms and standard library capabilities documented in `.github/instructions/go.instructions.md`.

## Mode & Scope Resolution

Parse the user's prompt arguments:

### Flag 1: `--current` *(Default mode)*
- Applies if `--current` is explicitly provided, or if no flag is specified and an active file is open in the editor.
- **Scope**: Target **only the currently active file or selected snippet**.
- Inspect the project's `go.mod` file to determine the target Go version.
- Scan the file for legacy or suboptimal patterns (e.g., manual loops instead of `slices.Contains`, custom comparison helpers instead of `min`/`max`, obsolete `v := v` loop bindings in Go 1.22+, `interface{}` instead of `any`).
- Present:
  1. A concise bulleted list of outdated patterns found.
  2. Clear **Before vs. After** diffs explaining each change.
  3. The fully refactored file code, ready to be accepted.

### Flag 2: `--full`
- Applies when the user passes `--full`.
- **Scope**: Audit the **entire workspace repository**.
- Step 1: Read `go.mod` to establish the target Go version constraint.
- Step 2: Scan all `.go` files across all packages.
- Step 3: Produce a structured modernization report:
  - **Summary**: Total modernization opportunities grouped by category (Collections, Concurrency, Error Handling, Control Flow).
  - **By Package & File**: List each file with its applicable rule IDs (e.g. `slices_contains`, `for_range_int`, `cmp_or`, `builtin_min_max`).
  - **Step-by-step Refactoring Plan**: Provide the code transformations prioritized by impact (cleanliness, performance, bug prevention).

---

## Modernization Checklist to Check

Always verify the target version in `go.mod` and apply only features valid for that version:

1. **Collections & Slices**:
   - `slices.Contains(s, v)` instead of manual search `for` loops (Go 1.21+)
   - `slices.Clone(s)` / `maps.Clone(m)` instead of manual allocation & copy loops (Go 1.21+)
   - `slices.Concat(s1, s2)` instead of chained `append` (Go 1.22+)
   - `slices.Chunk(s, size)` instead of manual slice index math (Go 1.23+)
   - `clear(s)` / `clear(m)` instead of manual wipe loops (Go 1.21+)
2. **Comparisons & Conditionals**:
   - Built-in `min(...)` and `max(...)` instead of custom comparison helpers or `if-else` (Go 1.21+)
   - `cmp.Or(a, b, c)` instead of chaining empty/nil checks (Go 1.21+)
3. **Loops & Iteration**:
   - `for i := range n` instead of `for i := 0; i < n; i++` (Go 1.22+)
   - Remove redundant `v := v` loop variable shadowing in Go 1.22+
   - `iter.Seq` / `iter.Seq2` for custom collection traversal (Go 1.23+)
4. **Testing & Benchmarking**:
   - `for b.Loop()` instead of `for i := 0; i < b.N; i++` in benchmarks (Go 1.24+)
   - `t.Context()` instead of manual `context.WithCancel` for test lifetimes (Go 1.24+)
5. **Errors & Context**:
   - `errors.Join(err1, err2)` instead of custom multierror structs (Go 1.20+)
   - `errors.AsType[T](err)` instead of uninitialized pointer targets (Go 1.26+)
   - `context.WithCancelCause` for cancellation reasons (Go 1.20+)
6. **Types & Pointers**:
   - `any` instead of `interface{}` (Go 1.18+)
   - `new(value)` for taking pointers to literal values (Go 1.26+)
   - `sync.WaitGroup.Go(fn)` instead of `wg.Add(1)` + `defer wg.Done()` (Go 1.24+)
   - Typed atomics (`atomic.Int64`, `atomic.Bool`) instead of `atomic.LoadInt64` (Go 1.18+)

---

## Output Format

- Explain the rationale for each modernization (e.g., standard library alignment, performance, readability).
- Ensure generated code strictly compiles with the version specified in `go.mod`.
- Ensure happy paths stay left-aligned and no error handling is lost or degraded.
