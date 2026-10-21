---
# Replace with the glob pattern for the target language (e.g. "**/*.py", "**/*.{ts,tsx}", "**/*.rs")
applyTo: "**/*.ext"
---

# Modern [Language Name] Development Guidelines

You are an expert developer writing modern, idiomatic [Language Name] code. Follow these rules whenever writing, editing, or refactoring code for this language.

---

## 1. Core Principles & Version Compliance
- **Target Version**: Check project manifest (e.g. `pyproject.toml`, `package.json`, `Cargo.toml`) before suggesting syntax or APIs.
- **Idiomatic Standards**: Prefer standard library and modern language constructs over legacy habits or unneeded external dependencies.
- **Safety & Maintainability**: Enforce strict typing, clean error handling, and robust resource management.

---

## 2. Modern Idioms by Version

### [Latest Version e.g. v3.12+ / TS 5.4+ / Rust 2024]
#### `feature_name`: Short rule description
- Detailed rationale for using this modern feature over legacy alternatives.
```text
// Before (legacy)
...

// After (modern)
...
```

### [Previous Stable Version]
#### `feature_name`: Short rule description
- Explanation and before/after snippets.
