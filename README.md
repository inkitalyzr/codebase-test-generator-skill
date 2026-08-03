# Codebase Test Generator

Cursor Agent Skill that analyzes a codebase and writes real, runnable automated tests for undertested or untested code — matching the project's existing language and test framework.

## What it does

- Scopes work to specific files, recent changes, or a PR/diff
- Detects the stack and existing test framework (or picks a sensible default)
- Reads source before writing tests; extends coverage instead of duplicating it
- Covers happy path, edge cases, error handling, and regression cases
- Runs the new tests when execution is available, and reports results

## Supported stacks

Language-specific conventions live under `references/`:


| Stack                   | Reference                             |
| ----------------------- | ------------------------------------- |
| Python                  | `references/python.md`                |
| JavaScript / TypeScript | `references/javascript-typescript.md` |
| Go                      | `references/go.md`                    |
| Java / Kotlin           | `references/java-kotlin.md`           |
| Other                   | `references/general-fallback.md`      |


## Install

**Personal skill** (available across projects):

```bash
cp -R . ~/.cursor/skills/codebase-test-generator
```

**Project skill** (shared with anyone using the repo):

```bash
mkdir -p .cursor/skills
cp -R /path/to/codebase-test-generator-skill .cursor/skills/codebase-test-generator
```

## Layout

```
.
├── SKILL.md              # Main skill instructions
├── references/           # Framework-specific conventions
└── README.md
```

## When to use it

Ask the agent to add tests, increase coverage, generate test cases, harden a module, or review gaps in an existing suite. The skill is triggered by those kinds of requests via its description in `SKILL.md`.