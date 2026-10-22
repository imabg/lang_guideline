# Global Repository Guidelines for GitHub Copilot

This repository adheres to modern, idiomatic engineering practices. Follow these general guidelines across all projects and languages.

## 1. Modular Language Instructions
Language-specific guidelines and modern idioms are stored in `.github/instructions/`:
- **Go files (`**/*.go`)**: Consult and strictly apply [`.github/instructions/go.instructions.md`](.github/instructions/go.instructions.md).
- When other languages are added, consult their matching `.instructions.md` file.

## 2. Core Coding Standards
1. **Target Version Adherence**:
   - Always verify the project's target language version before suggesting code (e.g., inspect `go.mod` for Go projects, `pyproject.toml` for Python, `package.json` for Node/TS).
   - Never introduce syntax, keywords, or APIs released after the target project's specified version.
2. **Left-Aligned Happy Path**:
   - Write defensive code that checks guards and errors early, returning or handling them immediately.
   - Avoid deep nesting and unnecessary `else` or `else if` blocks.
3. **Standard Library Over External Dependencies**:
   - Maximize standard library capabilities (e.g., standard collections, routing, serialization, structured logging).
   - Avoid recommending external third-party utility libraries for tasks natively solved in the modern standard library.
4. **Context & Safety**:
   - Always propagate context (`context.Context` in Go) across asynchronous boundaries and network calls.
   - Do not leak resources, goroutines, or open file descriptors; always pair initialization with clean teardown (e.g., `defer`).

## 3. Available Copilot Prompts & Incremental Action Files
Users can invoke specialized prompt commands in Copilot Chat:
- `/modernize-go [--current | --full]`: Scans and modernizes Go code to modern idioms (current file by default, or entire project with `--full`).
- `/review-go`: Performs a thorough modern Go code review against code changes.

For autonomous agent execution, 3 independent, incremental action files are available in `actions/`:
1. `actions/01-go-core-data-structures.action.md` (Phase 1: Core language, collections, iterations, built-ins)
2. `actions/02-go-concurrency-systems.action.md` (Phase 2: Concurrency, atomics, networking, HTTP routing, sandboxed I/O)
3. `actions/03-go-errors-testing-observability.action.md` (Phase 3: Error resilience, modern testing, benchmarks, slog)
