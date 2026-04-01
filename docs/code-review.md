# Code Review

A structured, security-first code review skill with testability assessment and clear merge recommendations.

## What It Does

When invoked, the skill automatically:

1. **Detects scope** — works with feature branches, uncommitted changes, specific files, or PRs
2. **Reads full file context** — analyzes complete files, not just diffs, so issues in surrounding code aren't missed
3. **Finds and reads related tests** — checks existing coverage before flagging gaps
4. **Reviews against 4 severity tiers** — Critical (security), High (will cause errors), Medium (code quality), Low (style)
5. **Assesses testability** — flags tight coupling, side effects, and missing DI seams with refactoring suggestions
6. **Evaluates test coverage** — identifies missing scenarios across happy path, edge cases, error paths, and integration points
7. **Produces a structured report** with findings table, verdict, and a clear merge recommendation (Block / Request Changes / Approve)

## Usage

```
/code-review
```

Or with a specific target branch:

```
/code-review against develop
```

## What Gets Checked

### Critical — Security (blocks merge)

Hardcoded secrets, SQL/command/template injection, XSS, path traversal, auth gaps, SSRF, insecure crypto, unsafe deserialization.

### High — Will Cause Errors

Null dereferences, race conditions, resource leaks, breaking API changes, off-by-one errors, type mismatches.

### Medium — Code Quality

Long functions (>50 lines), large files (>500 lines), deep nesting (>4 levels), dead code, missing error handling, missing timeouts, unnecessary complexity.

### Low — Style

Debug logging, unlinked TODOs, inconsistent patterns. Only flagged when meaningful.

## Output Example

The skill produces a structured report:

```
Changed files summary:
| File              | Lines +/- | Risk   |
|-------------------|-----------|--------|
| src/auth.py       | +42 / -3  | Critical |
| src/utils/hash.py | +15 / -8  | Medium |

Findings (Critical first):
| Severity | File:Line        | Issue                          | Fix                      |
|----------|------------------|--------------------------------|--------------------------|
| Critical | src/auth.py:42   | SQL built with f-string        | Use parameterized query  |
| Medium   | src/utils/hash.py:23 | Empty catch block          | Log error or re-raise    |

Testability & Coverage:
| Aspect         | Assessment                              |
|----------------|-----------------------------------------|
| Testability    | Needs Refactoring — DB client hardcoded |
| Tests added?   | None                                    |
| Coverage gaps  | No error path tests for auth flow       |
| Existing tests | Still passing                           |

Verdict:
| Category      | Result |
|---------------|--------|
| Security      | FAIL   |
| Errors        | No     |
| Code quality  | Moderate |
| Test coverage | Gaps   |

Recommendation: **Block** — fix SQL injection in auth.py before merge.

Action items:
1. Use parameterized queries in src/auth.py:42
2. Add error handling in src/utils/hash.py:23
3. Add tests for auth error paths
```

## Configuration

The skill defaults to comparing against `main`. To change the default target branch, edit the skill file:

```markdown
- **Feature branch**: diff against target branch (default: `main`, ...
```

Quality thresholds can also be adjusted in the Medium — Code Quality section:

| Threshold | Default | Where to change |
|-----------|---------|-----------------|
| Max function length | 50 lines | `Functions > 50 lines` |
| Max file length | 500 lines | `files > 500 lines` |
| Max nesting depth | 4 levels | `nesting > 4 levels` |

## Language Support

The skill is language-agnostic. It identifies the language/framework of each changed file and applies appropriate checks. Works with any language Claude supports.
