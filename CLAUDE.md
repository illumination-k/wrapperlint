# CLAUDE.md

## Commands

Run `mise install` first to install all tools.

```bash
mise run ci    # Run all ci:* tasks
mise run fmt   # Run all fmt:* tasks
mise run lint  # Run all lint:* tasks
mise run test  # Run all test:* tasks
```

## Tools

All tools are managed by mise. Run `mise install` to install them.

| Tool          | Purpose                                 |
| ------------- | --------------------------------------- |
| uv            | Python package manager                  |
| dprint        | Code formatter                          |
| prek          | Pre-commit hook runner                  |
| shfmt         | Shell script formatter                  |
| actionlint    | GitHub Actions linter                   |
| zizmor        | GitHub Actions security linter          |
| shellcheck    | Shell script linter                     |
| ghalint       | GitHub Actions linter                   |
| pinact        | Pin GitHub Actions versions to SHAs     |
| rust          | Rust toolchain                          |
| cargo-nextest | Fast Rust test runner                   |
| cargo-deny    | Dependency license and advisory checker |
| cargo-audit   | Security advisory checker for Rust      |

## Purpose

wrapperlint is a linter for detecting low-value wrapper functions: functions that mostly forward to another function, add little semantic value, and are not reused enough to justify the abstraction. The initial target is Oxlint/Oxc in Rust, with the domain logic separated for future backend reuse.

## Architecture

### Workspace Structure

Rust workspace with two crates:

- `crates/wrapperlint-domain/` — AST-independent scoring logic, feature structs, policy thresholds, and evaluation decisions. No Oxc types or parser logic.
- `crates/wrapperlint-oxlint/` — Oxlint/Oxc-specific AST traversal, feature extraction, call-site counting, diagnostics, and rule registration. Depends on `wrapperlint-domain`.

### Detection Model

The rule `no-trivial-wrapper` flags functions that are jointly:
1. **Thin** — small statement count, single call, no branching/loops
2. **Low semantic delta** — no validation, error handling, transforms, or policy injection
3. **Low reuse** — few call sites, single caller, same-file only

### Key Design Constraints

- Do NOT introduce generic AST abstractions in the domain crate (no custom FunctionNode traits, no unified AST hierarchy)
- Backend extracts features → domain scores features
- Same-file reuse counting only in MVP
- Exported functions are exempt in MVP
- Diagnostics must explain why the wrapper was flagged
