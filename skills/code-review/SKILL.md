---
name: code-review
description: "Thorough code review of uncommitted changes or current branch vs. a target branch"
---

# Code Review

## 1. Determine Scope

```
git branch --show-current
git status --short
```

- **Feature branch**: diff against target branch (default: `main`, or as specified by user).
- **Uncommitted changes**: diff staged + unstaged against HEAD.
- **User specifies files or PR**: review that scope.

## 2. Read Full Context

For every changed file, read the **complete file** — not just the diff. Changes must be understood in context. Identify the language/framework to apply appropriate checks.

## 3. Read Related Tests

Find test files for changed code (`*_test.*`, `*.test.*`, `*.spec.*`, or under `tests/`/`__tests__/`). Read them. Note whether tests were added or modified in this changeset.

## 4. Review Against These Categories

### Critical — Security (blocks merge)

- Hardcoded secrets (API keys, tokens, passwords, private keys)
- Injection (SQL, command, template, LDAP)
- XSS — unsanitized user input in HTML/DOM
- Path traversal — user input in file paths
- Auth gaps — missing authn/authz on endpoints
- SSRF — user-controlled URLs in server-side requests
- Insecure crypto (MD5/SHA1 for passwords, weak random)
- Unsafe deserialization of untrusted data

### High — Will Cause Errors

- Null/undefined dereferences, unhandled exceptions
- Race conditions, deadlocks, resource leaks
- Breaking changes to APIs/contracts without migration
- Config that will fail in certain environments
- Off-by-one errors, wrong boundary conditions
- Type mismatches that fail at runtime

### Medium — Code Quality

- Functions > 50 lines, files > 500 lines, nesting > 4 levels
- Hardcoded magic values that should be constants/config
- Dead code (commented-out code, unused imports, unreachable branches)
- Missing error handling or empty catch blocks
- Missing timeouts on network/external calls
- Poor naming, unclear separation of concerns
- Over-abstraction or unnecessary complexity

### Low — Style

Only flag if meaningful: debug logging left in, TODO/FIXME without issue links, inconsistent patterns that will confuse future readers.

## 5. Assess Testability

Evaluate whether the changed code is structured for effective testing:

- **Dependency injection**: Can dependencies be swapped with test doubles, or are they hardcoded/tightly coupled?
- **Side effects**: Does object creation or initialization trigger network calls, file I/O, or other side effects that block unit testing?
- **Pure functions**: Are computations separated from side effects where possible?
- **Test seams**: Are there clear boundaries where mocks/stubs can be inserted?

If testability is poor, include specific refactoring suggestions (extract interface, inject dependency, separate pure logic from I/O).

## 6. Assess Test Coverage

- **Tests added/modified?** List them explicitly. If no tests were added for new functionality, flag it.
- **Missing scenarios** — identify gaps in:
  - Happy path: Does the basic intended use case have coverage?
  - Edge cases: Boundary values, empty inputs, max limits
  - Error paths: Invalid input, network failures, timeouts, permission denied
  - State transitions: If the code changes state, are transitions tested?
- **Broken tests**: Will existing tests still pass with these changes, or do they need updating?
- **Integration points**: Are interactions with external services, databases, or APIs tested (or at least mockable)?

## 7. Output

**Changed files summary:**

| File | Lines +/- | Risk |
|---|---|---|
| `path/to/file` | +X / -Y | Critical/High/Medium/Low |

**Findings** (grouped by severity, Critical first):

| Severity | File:Line | Issue | Fix |
|---|---|---|---|
| Critical | `src/auth.py:42` | SQL built with f-string | Use parameterized query |

**Testability & Coverage:**

| Aspect | Assessment |
|---|---|
| Testability | Good / Needs Refactoring — brief explanation |
| Tests added? | Yes / Partial / None |
| Coverage gaps | List missing scenarios |
| Existing tests | Still passing / Likely broken |

**Verdict:**

| Category | Result |
|---|---|
| Security | Pass / FAIL |
| Errors | Yes / No |
| Code quality | High / Moderate / Low |
| Test coverage | Good / Gaps / None |

**Recommendation**: **Block** (must fix), **Request Changes** (should fix), or **Approve** (optional improvements only). End with a prioritized list of action items.
