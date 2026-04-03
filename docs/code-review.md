# Code Review

Two skills for different levels of review depth — both security-first with clear merge recommendations.

| Skill | When to use |
|-------|------------|
| `/code-review` | Normal development — fast, structured, covers security + quality + testability |
| `/ult-code-review` | High-stakes changes — 10 parallel specialist agents, triple correctness passes, systematic coverage gap analysis |

For checking code against *your project's specific rules*, see [Project Guidelines](#project-guidelines) at the bottom.

---

## /code-review

### What It Does

When invoked, the skill automatically:

1. **Detects scope** — works with feature branches, uncommitted changes, specific files, or PRs
2. **Reads full file context** — analyzes complete files, not just diffs, so issues in surrounding code aren't missed
3. **Finds and reads related tests** — checks existing coverage before flagging gaps
4. **Reviews against 4 severity tiers** — Critical (security), High (will cause errors), Medium (code quality), Low (style)
5. **Assesses testability** — flags tight coupling, side effects, and missing DI seams with refactoring suggestions
6. **Evaluates test coverage** — identifies missing scenarios across happy path, edge cases, error paths, and integration points
7. **Produces a structured report** with findings table, verdict, and a clear merge recommendation (Block / Request Changes / Approve)

### Usage

```
/code-review
```

Or with a specific target branch:

```
/code-review against develop
```

### What Gets Checked

#### Critical — Security (blocks merge)

Hardcoded secrets, SQL/command/template injection, XSS, path traversal, auth gaps, SSRF, insecure crypto, unsafe deserialization.

#### High — Will Cause Errors

Null dereferences, race conditions, resource leaks, breaking API changes, off-by-one errors, type mismatches.

#### Medium — Code Quality

Long functions (>50 lines), large files (>500 lines), deep nesting (>4 levels), dead code, missing error handling, missing timeouts, unnecessary complexity.

#### Low — Style

Debug logging, unlinked TODOs, inconsistent patterns. Only flagged when meaningful.

### Output Example

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

### Configuration

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

### Language Support

The skill is language-agnostic. It identifies the language/framework of each changed file and applies appropriate checks. Works with any language Claude supports.

---

## /ult-code-review

10 parallel specialist agents running simultaneously for near-complete bug and vulnerability coverage.

### What It Does

1. **Detects scope** — same as `/code-review`: uncommitted changes, branch diff, specific files, or PRs
2. **Reads full file context** — passes complete files (not just diffs) to every agent
3. **Launches 10 agents in parallel** — each covering a distinct review dimension simultaneously
4. **Runs triple-pass correctness analysis** — forward logic trace, adversarial inputs, and edge cases independently
5. **Runs systematic coverage gap analysis** — enumerates all code paths, maps existing tests, reports untested paths sorted by risk
6. **Synthesizes findings** — deduplicates across agents, boosts priority when multiple agents flag the same issue

### The 10 Agents

| Agent | Focus |
|-------|-------|
| Security | Secrets, injection, XSS, path traversal, auth gaps, SSRF, insecure crypto, deserialization |
| Architecture & Design | SRP, coupling, cohesion, abstraction, API design, SOLID violations |
| Performance | Algorithmic complexity, N+1 queries, unbounded operations, memory leaks, blocking async |
| Error Handling | Uncaught exceptions, silent swallowing, missing propagation, timeouts, logging gaps |
| Dependencies | Known CVEs, outdated packages, unused imports, license issues, supply chain |
| Language Idioms | Anti-patterns, deprecated APIs, type safety gaps, framework conventions |
| Correctness A — Forward | Traces each function's execution path and verifies it produces correct output |
| Correctness B — Adversarial | Attempts to break code with null, empty, negative, MAX_INT, out-of-order inputs |
| Correctness C — Edge Cases | Off-by-one, integer overflow, empty collections, timezone/locale/encoding edge cases |
| Coverage — Gap Analysis | Enumerates all code paths, maps existing tests, reports every untested path by risk |

### Usage

```
/ult-code-review
```

Or with a specific scope:

```
/ult-code-review --branch       # Current branch vs main
/ult-code-review --pr           # PR diff
/ult-code-review src/auth.py    # Single file
/ult-code-review src/           # Directory
```

### Output

The report has four sections: **Changed Files**, **Findings** (all agents merged and ranked), **Correctness Verdict** (three-pass summary with confidence score), and **Coverage Verdict** (path count, coverage %, top gaps with suggested tests).

---

## Project Guidelines

Both `/code-review` and `/ult-code-review` apply universal best practices. To also check code against *your project's specific rules* — naming conventions, architectural constraints, stack-specific standards — use `/guidelines-review`.

The two are complementary: code review catches bugs and security issues; guidelines review enforces the standards your team has agreed on. See the [Project Guidelines doc](guidelines.md) for how to define and use them.
