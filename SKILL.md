---
name: codebase-test-generator
description: Analyzes a codebase (any language/framework) and writes new automated test cases for undertested or untested code. Use this whenever the user asks to "add tests", "write unit tests", "increase test coverage", "generate test cases", "test this function/module/file", or mentions a codebase lacking tests — even if they don't say the word "test" explicitly (e.g. "make sure this is solid before I ship it", "harden this module"). Also use for reviewing/improving an existing test suite, writing regression tests for a bug fix, or auditing coverage gaps. Do NOT use for writing a single trivial test the user has fully specified themselves (e.g. "test that add(2,3) returns 5") — just write it inline.
---

# Codebase Test Generator

Generates real, runnable, well-scoped test cases for an existing codebase, in whatever
language and test framework the project already uses (or a sensible default if there is none yet).

## Workflow

### Step 0 — Determine execution mode

Check whether you have `bash_tool` / shell execution available in this session.

- **Can execute code** (Claude Code, Cowork, or claude.ai with the code-execution tool on):
  full workflow below, including actually *running* the generated tests.
- **Cannot execute code** (chat-only, no sandbox): do steps 1–4 below, generate the test
  file(s), but say clearly that you were not able to execute them, and ask the user to run
  the suite and paste back any failures so you can fix them.

Never skip execution silently — if you can run tests, always do so before presenting them
as done.

### Step 1 — Scope the task

"Any codebase" is too broad to attack all at once. Before writing anything, pin down scope:

- If the user named specific file(s)/function(s) → scope to those.
- If the user said "this repo" / "this project" with no further detail → default to the
  **most recently changed files** (`git diff --stat HEAD~5` or the equivalent) or ask which
  module/directory to prioritize if the repo is large (>~30 source files) and nothing else
  narrows it down.
- If there's an explicit diff / PR / uncommitted changes → default to testing *that*, since
  it's almost always what "add tests" means in context.

State the scope you're using in one line before proceeding, e.g. "Scoping this to
`src/billing/` since that's what changed most recently."

### Step 2 — Detect the stack

Identify, in order:

1. **Language(s)** present (file extensions).
2. **Existing test framework**, from config/lockfiles and existing test directories:
   - Python → `pytest.ini`, `pyproject.toml` `[tool.pytest]`, `unittest` imports
   - JS/TS → `package.json` `devDependencies` (jest, vitest, mocha)
   - Go → `_test.go` files (stdlib `testing`, or `testify`)
   - Java/Kotlin → `pom.xml`/`build.gradle` (JUnit4/5, TestNG)
   - Ruby → `Gemfile` (RSpec, Minitest)
   - Rust → Cargo, built-in `#[test]`
3. If **no test framework exists yet**, pick the ecosystem-standard default (pytest for
   Python, Jest for a plain JS project / the framework's native test runner if it's a
   framework like Next.js or Rails, `go test` for Go, JUnit5 for Java) and say so — don't
   silently invent a bespoke one.

Load the matching reference file from `references/` for framework-specific conventions
(assertion style, mocking library, fixture/setup patterns, file naming and location):

- `references/python.md`
- `references/javascript-typescript.md`
- `references/go.md`
- `references/java-kotlin.md`
- `references/general-fallback.md` — for any language/framework not covered above; apply
  the same principles generically.

### Step 3 — Read before writing

- Read the actual source of the function/module/file being tested — never guess its
  behavior from its name.
- Check for existing tests covering the same code (search test dirs for the function/class
  name) so you extend coverage instead of duplicating it.
- Note the function's real dependencies (DB calls, network, filesystem, time, randomness) —
  this determines what needs mocking.
- If available, check recent git history / commit messages for the file: a recent bug fix
  with no accompanying test is a strong signal to write a regression test for it.

### Step 4 — Write the tests

For each unit under test, aim to cover, roughly in this priority order:

1. **Happy path** — typical valid input, expected output.
2. **Boundary/edge cases** — empty input, zero, negative numbers, max length, single-element
   collections, Unicode/special characters where relevant.
3. **Error handling** — invalid input, expected exceptions/error returns, timeouts.
4. **Security-relevant boundary cases** — only for functions that touch a risky surface:
   building a query/command/template from input, constructing a filesystem path from
   input, or gating auth/access. For those specifically, add targeted tests for:
   - Injection-style inputs (SQL/command/template strings) where input flows into a
     query, command, or template.
   - Path traversal inputs (`../`, absolute paths) where a user-influenced path touches
     the filesystem.
   - Auth-bypass-shaped inputs (empty/null tokens, malformed roles) where the function
     gates access.

   **Reachability gate:** only generate this tier if the function actually exercises one
   of these risky patterns — do not add generic security tests to code with no such
   surface (e.g. a pure currency-formatting function gets none of these). This keeps the
   tier high-signal rather than padding, and mirrors how dedicated tools like Snyk use
   reachability analysis to cut noise. This tier complements a real SAST/security
   scanner — it is not a substitute for one, and should stay scoped to a small number of
   sharp tests, not an exhaustive attack-surface sweep.
5. **Regression case** — if step 3 turned up a bug-fix commit without a test.

Skip low-value tests: trivial getters/setters, framework boilerplate, purely declarative
config, or code that's about to be deleted/refactored per the user's stated plans. Flag
these as skipped rather than writing filler tests just to inflate a count.

Mocking guidance (expand per-framework in the reference files):
- Mock true externalities (network, DB, filesystem, wall-clock time, randomness).
- Don't mock the thing you're actually testing, and don't over-mock internal collaborators
  to the point the test just checks that mocks were called — assert on real behavior/output
  wherever feasible.

Follow the project's existing test file naming/location convention (e.g. `test_foo.py`
next to `foo.py`, or `foo.test.ts` next to `foo.ts`, or a mirrored `tests/` tree) rather than
inventing a new convention.

### Step 5 — Run and verify (execution-capable environments only)

- Actually run the new test file(s), not just the new tests in isolation — check you haven't
  broken anything adjacent.
- If a test **you wrote** fails because of a mistake in the test (wrong expected value, bad
  mock setup) — fix it and rerun.
- If a test fails because it **exposes a real bug** in the source code — do NOT edit the
  source to make the test pass. Leave the test failing, and clearly flag it to the user as
  a likely genuine bug, with the failure output.
- If a coverage tool is available and cheap to run (`pytest --cov`, `nyc`/`c8`, `go test
  -cover`, `jacoco`), report the before/after coverage delta for the scoped files.

### Step 6 — Report back

Summarize: what was tested, what was intentionally skipped and why, how many tests
pass/fail, any real bugs surfaced, and the coverage delta if measured. Keep this concise —
a short list, not a essay.

## Principles

- **Real assertions over vanity tests.** A test that always passes regardless of the code's
  behavior (e.g. `assert result is not None` when more precision is possible) is worse than
  no test — it creates false confidence.
- **Don't rewrite the user's source to fit your tests.** Tests conform to the code's actual
  contract; if the contract seems wrong, say so instead of quietly "fixing" it.
- **Prefer fewer, sharper tests over exhaustive combinatorial ones**, unless the function is
  genuinely combinatorial (e.g. a parser, validator) where table-driven tests earn their keep.
- **Match existing style.** Read a couple of the project's existing tests (if any) and mirror
  naming, assertion style, and structure before introducing your own conventions.
