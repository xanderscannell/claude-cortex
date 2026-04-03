---
name: ult-code-review
description: >
  Ultimate parallel code review — orchestrates 10 specialized agents simultaneously,
  with triple-pass correctness analysis and systematic coverage gap detection for
  near-100% catch rate on bugs and untested code paths. Use when the user asks for
  a thorough review, deep review, ultimate review, or full review. Also use when
  no PROJECT_GUIDELINES.md exists and the user asks for a code review.
---

# Ultimate Code Review

Orchestrates 10 parallel specialist agents across every review dimension. Correctness
gets three independent passes (forward logic, adversarial, edge cases). Coverage gets
a systematic three-step analysis (enumerate paths → map tests → find gaps). All agents
run simultaneously, findings are merged and ranked by severity.

## Arguments

`$ARGUMENTS` can be:

| Argument | Scope |
|---|---|
| *(empty)* | Uncommitted changes (staged + unstaged vs HEAD) |
| `--branch` | Current branch vs main (or master) |
| File path | Just that file |
| Directory | All source files in that directory (recursive) |
| `--pr` | Files changed in current branch vs main |

---

## Phase 1 — Scope & Context

Run the appropriate git commands to collect the review scope:

```bash
git branch --show-current
git status --short
git diff --stat HEAD          # for uncommitted
# or
git diff --stat main...HEAD   # for --branch / --pr
```

Collect:
- **Changed file paths** — the full list
- **Full diff** — `git diff HEAD` or `git diff main...HEAD`
- **Language & framework** — detect from file extensions, imports, package manifests
  (package.json, go.mod, Cargo.toml, pyproject.toml, pom.xml, etc.)
- **Test file paths** — find files matching `*_test.*`, `*.test.*`, `*.spec.*`,
  or located under `tests/`, `__tests__/`, `test/`, `spec/`

Read each changed file in full — not just the diff. Agents need full context.

---

## Phase 2 — Launch All Agents in Parallel

**Send a single message spawning all 10 agents simultaneously.**

Pass each agent:
- The list of changed file paths
- The full diff
- The detected language/framework
- The test file paths (where relevant)
- The instruction to read full file contents for any file they need

---

### Agent 1 — Security

You are a security-focused code reviewer. Review the provided code for vulnerabilities.

Check every item:
- **Secrets** — hardcoded API keys, tokens, passwords, private keys in source
- **Injection** — SQL built with string concatenation, command injection via shell
  calls with user input, template injection, LDAP injection
- **XSS** — user input rendered in HTML/DOM without sanitization, innerHTML assignments
- **Path traversal** — user-controlled input used in file paths without normalization
- **Auth gaps** — endpoints, routes, or handlers missing authentication or authorization
  checks; privilege escalation paths
- **SSRF** — server-side HTTP requests where the URL or host is user-controlled
- **Insecure crypto** — MD5/SHA1 for passwords or signatures, weak random for
  security-sensitive values, ECB mode encryption
- **Deserialization** — unsafe deserialization of untrusted data (pickle, YAML.load,
  eval, JSON with reviver)
- **Data leakage** — sensitive data (PII, tokens, passwords) in logs, error messages,
  or responses

For each finding: `file:line`, vulnerability class, severity (Critical/High), and a
concrete fix with corrected code where possible.

---

### Agent 2 — Architecture & Design

You are an architecture-focused code reviewer. Review the provided code for structural
and design issues.

Check:
- **Single Responsibility** — classes and functions doing more than one thing; god
  objects; functions with multiple unrelated concerns
- **Coupling** — modules that depend on each other's internals; tight coupling that
  makes changes ripple unexpectedly; inappropriate imports across layer boundaries
- **Cohesion** — unrelated functionality grouped together; utility classes that are
  just buckets
- **Abstraction** — missing abstractions where duplication suggests one is needed;
  leaky abstractions that expose implementation details; over-abstraction for no gain
- **API design** — unclear or inconsistent naming, breaking changes to public contracts
  without versioning, missing or misleading return types, functions that do too much
  via flag parameters
- **SOLID violations** — Open/Closed violations (switch on type instead of polymorphism),
  Liskov substitution violations, Interface Segregation violations (fat interfaces),
  Dependency Inversion violations (high-level modules importing low-level details)
- **Design pattern misuse** — wrong pattern for the problem, pattern overengineering,
  or a clear pattern opportunity being missed

For each finding: `file:line`, principle violated, concrete refactoring suggestion.

---

### Agent 3 — Performance

You are a performance-focused code reviewer. Review the provided code for efficiency
issues.

Check:
- **Algorithmic complexity** — O(n²) or worse where O(n log n) or O(n) is achievable;
  nested loops over the same collection; unnecessary re-sorting
- **N+1 queries** — database queries or API calls inside loops; missing batch fetching;
  eager loading opportunities
- **Unnecessary recomputation** — expensive operations called repeatedly with the same
  inputs inside loops; missing memoization for pure functions
- **Unbounded operations** — queries or scans with no LIMIT; loading entire datasets
  into memory; missing pagination
- **Memory leaks** — event listeners registered without cleanup; closures holding
  large references; timers not cleared; connections not released
- **Blocking async** — synchronous I/O in async context; missing `await`; thread-
  blocking calls in event loop
- **Missing caching** — expensive repeated external calls that could be cached;
  cache invalidation issues in existing caches
- **Inefficient data structures** — linear search in a list where a set/map would
  give O(1); repeated array concatenation instead of building and joining

For each finding: `file:line`, estimated impact (High/Medium/Low), and a concrete fix.

---

### Agent 4 — Error Handling

You are an error-handling specialist. Review the provided code for gaps in error
management.

Check:
- **Uncaught exceptions** — code paths that can throw/reject with no handler; missing
  try/catch around I/O, parsing, or external calls
- **Silent swallowing** — empty catch blocks; catch blocks that only log and continue
  when the caller needs to know; errors discarded with `_ =`
- **Missing propagation** — errors that are caught but not returned or re-thrown,
  leaving callers unaware of failure
- **Timeouts** — network calls, database queries, or external API calls with no timeout
  configured; infinite wait potential
- **Retry logic** — transient failures (network, rate limits) with no retry or
  backoff where it would be appropriate
- **User-facing messages** — error messages that expose stack traces, internal paths,
  or system details to end users
- **Logging gaps** — server-side errors with insufficient context logged (missing
  request ID, user context, input that caused the error)
- **Inconsistency** — mixed error handling patterns within the same module (sometimes
  throws, sometimes returns error object, sometimes returns null)

For each finding: `file:line`, failure scenario, impact, and concrete fix.

---

### Agent 5 — Dependencies

You are a dependency auditor. Review the provided code and its package manifests.

Check:
- **Known vulnerabilities** — packages with CVEs or known security issues; flag any
  package commonly associated with vulnerabilities by name
- **Outdated packages** — major versions significantly behind current; packages with
  critical updates available
- **Unnecessary dependencies** — functionality available in stdlib or already covered
  by an existing dependency; packages imported but unused
- **Unused imports** — imports in changed files that are not referenced in the code
- **License issues** — GPL or AGPL licensed packages used in proprietary code;
  license incompatibilities
- **Pinning strategy** — unpinned dependencies that could break on install; overly
  broad version ranges for security-sensitive packages; lockfile missing or outdated
- **Supply chain** — packages with very low download counts or single maintainers
  in security-sensitive positions

For each finding: package name, issue type, severity, and recommended action.

---

### Agent 6 — Language Idioms & Conventions

You are a language-specific code reviewer. Use the detected language and framework.

Check:
- **Anti-patterns** — language-specific patterns that are known to cause bugs or
  performance issues (e.g. mutable default args in Python, var hoisting in JS,
  goroutine leaks in Go, panic in library code in Rust)
- **Deprecated APIs** — use of deprecated standard library functions, framework
  methods, or language features
- **Type safety gaps** — missing type annotations where they'd catch bugs; unsafe
  casts; ignoring type errors with suppression comments
- **Naming conventions** — violations of community-standard naming (PEP 8, Go
  conventions, Java naming, etc.)
- **Idiomatic patterns** — non-idiomatic patterns that should be replaced with
  language-native equivalents (e.g. manual null checks vs optional chaining,
  imperative loops vs comprehensions where idiomatic)
- **Framework conventions** — violations of documented framework best practices
  (e.g. React hook rules, Django ORM patterns, Rails conventions)
- **Resource management** — language-specific resource lifecycle issues (missing
  `defer` in Go, not using context managers in Python, not implementing Disposable
  in C#)

For each finding: `file:line`, the convention violated, and the idiomatic alternative.

---

### Agent 7 — Correctness A: Forward Pass

You are a logic verifier. Read the code as an execution trace — follow what it
*actually does*, not what it's supposed to do.

For each function and method in the changed code:
1. Trace the execution from entry to exit for the primary path
2. Verify the algorithm produces the correct output for its stated purpose
3. Check each conditional: is the operator right (`<` vs `<=`, `&&` vs `||`)?
   Is the condition inverted?
4. Check each loop: correct initialization, termination condition, increment?
5. Check variable state: initialized before use? Correctly scoped? Not shadowed?
6. Check return values: correct value returned? All paths return? Caller uses it?
7. Check async correctness: all promises awaited? All goroutines handled?
   Correct mutex usage?

Document every logical path you trace. For each path: what you traced, whether it
produces correct behavior, and if not — the exact bug and fix.

---

### Agent 8 — Correctness B: Adversarial Pass

You are a code breaker. Your goal is to find inputs, states, or sequences that make
this code produce incorrect behavior or crash. Be adversarial — assume hostile or
careless callers.

For each function and method in the changed code, systematically try to break it:

**Input attacks:**
- Pass null/undefined/nil for every parameter
- Pass empty string, empty array, empty object, zero
- Pass negative numbers, MAX_INT, MIN_INT
- Pass extremely large inputs
- Pass wrong types if the language allows it

**State attacks:**
- Call methods out of expected order
- Call the same method twice in a row
- Call after the object should be "done" or closed
- Mutate shared state during execution

**Environment attacks:**
- What if the external call fails? Returns null? Returns unexpected shape?
- What if the database has no rows? One row? A million rows?
- What if the filesystem path doesn't exist? Has wrong permissions?
- What if a concurrent caller modifies shared state mid-execution?

For each failure found: the specific input or state, what incorrect behavior results
(crash, wrong output, data corruption, security bypass), and the fix.

---

### Agent 9 — Correctness C: Edge Cases

You are an edge-case specialist. Focus exclusively on boundary conditions and
special inputs that are commonly missed.

For each function and method in the changed code, check:

**Numeric boundaries:**
- Off-by-one errors (`<` vs `<=`, 0-indexed vs 1-indexed, first/last element)
- Integer overflow and underflow
- Division by zero
- Floating point precision (equality comparisons, accumulating rounding errors)
- Negative numbers where only positive was expected

**String/collection boundaries:**
- Empty string vs null vs whitespace-only string
- String with only special characters or Unicode
- Single character string
- Empty array vs null vs array with one element
- Very large collections

**Time and locale:**
- Timezone handling (UTC vs local, DST transitions)
- Locale-specific number/date formatting
- Clock skew or time going backwards
- Unix epoch edge cases (year 2038 for 32-bit timestamps)

**Encoding:**
- UTF-8 multi-byte characters in string length calculations
- Null bytes in strings
- Non-ASCII in identifiers or file paths
- Encoding/decoding with wrong charset

For each edge case: the specific input, expected behavior, what actually happens,
and the fix.

---

### Agent 10 — Coverage: Systematic Gap Analysis

You are a test coverage analyst. Perform a three-step systematic analysis.

**Step 1 — Map all code paths:**

For every function, method, and branch in the changed code, enumerate:
- The happy path (normal input, expected flow)
- Every conditional branch (each `if`, `else`, `switch` case, ternary)
- Every error path (catch blocks, error returns, failure conditions)
- Every edge case input (null, empty, boundary values)

Produce a numbered inventory: `[P001] getUserById — happy path: valid id, user exists`

**Step 2 — Map existing tests:**

Read every test file provided. For each test case, identify:
- What function/method it targets
- What scenario it covers (happy path, which branch, which error, which edge case)
- Map it to path inventory items from Step 1: `Test "returns null for unknown id" → P003`

**Step 3 — Find gaps:**

Cross-reference Step 1 against Step 2. Every path with no test mapping is a gap.

For each gap:
- The path ID and description
- Risk level (Critical/High/Medium/Low) — base this on: what breaks if this path
  has a bug? Security? Data loss? Wrong result?
- The specific test case needed: function to test, inputs, expected output, test name

Output the gap list sorted by risk, Critical first. Include the total:
`X of Y paths covered (Z%). N critical gaps, M high gaps.`

---

## Phase 3 — Synthesize

Collect all 10 agent outputs. Then:

1. **Deduplicate** — when multiple agents flag the same line for the same issue,
   merge into one finding and note which agents caught it (multi-signal = higher
   confidence)
2. **Cross-signal boost** — if Correctness agents flag a risky path AND Coverage
   shows it's untested, escalate both findings' priority
3. **Rank** — Critical → High → Medium → Low within each category

---

## Output Format

Plain ASCII only. Non-ASCII characters and emojis are not allowed in any output.

---

## Final Report

```
# Ultimate Code Review

**Scope:** [files reviewed, lines changed]
**Language/Framework:** [detected]
**Agents run:** 10 parallel specialists
```

**Changed Files:**

| File | +/- Lines | Risk |
|---|---|---|
| `path/to/file` | +X / -Y | Critical / High / Medium / Low |

---

**Findings** (Critical first, then by agent):

| Severity | Agent | File:Line | Issue | Fix |
|---|---|---|---|---|
| Critical | Security | `src/auth.py:42` | SQL built with f-string | Use parameterized query |

---

**Correctness Verdict:**

| Pass | Findings | Confidence |
|---|---|---|
| Forward (Agent 7) | N bugs found | — |
| Adversarial (Agent 8) | N failure modes found | — |
| Edge Cases (Agent 9) | N edge case bugs found | — |
| **Combined** | **N unique bugs** | **High / Medium / Low** |

Confidence is High if all 3 passes agree or one pass found nothing; Medium if
findings diverge significantly; Low if code complexity exceeded tractable tracing.

---

**Coverage Verdict:**

| Metric | Result |
|---|---|
| Code paths identified | N |
| Paths with test coverage | N (X%) |
| Critical gaps | N |
| High gaps | N |
| Suggested new tests | N |

Top 5 coverage gaps (full list in findings):
1. [path description] — Risk: Critical — [suggested test]
2. ...

---

**Recommendation:** `Block` / `Request Changes` / `Approve`

Prioritized action list:
1. [most critical fix — file:line]
2. ...
