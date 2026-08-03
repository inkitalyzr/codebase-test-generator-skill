# JavaScript / TypeScript

## Framework detection
Check `package.json` `devDependencies`/`scripts`:
- `jest` or `ts-jest` → Jest
- `vitest` → Vitest (near-identical API to Jest, prefer if present)
- `mocha` + `chai` → Mocha/Chai
- Framework-native: Next.js/React → often Jest + React Testing Library; NestJS → Jest is
  built in; Angular → Jasmine/Karma or Jest depending on setup.

If none exists, default to **Vitest** for a modern ESM/TS project, **Jest** otherwise — check
which is faster to wire up given existing `tsconfig`/babel config.

## File placement & naming
- `<name>.test.ts` / `<name>.spec.ts` colocated with source, OR a mirrored `__tests__/` /
  `test/` directory — match whatever the repo already does.

## Style
- `describe`/`it` (or `test`) blocks; one `describe` per unit, one `it` per behavior, not per
  input — group boundary cases with `it.each` / `test.each` table-driven syntax instead of
  many near-duplicate `it` blocks.
- Prefer `toEqual`/`toStrictEqual` over `toBe` for object/array comparisons.
- For async code, always `await` assertions or return the promise — a dangling unawaited
  assertion silently passes even on failure.

## Mocking
- `jest.mock()` / `vi.mock()` for modules; `jest.spyOn` for partial mocks.
- Mock network calls with `msw` if already a dependency, otherwise mock the fetch/axios call
  directly.
- Use fake timers (`jest.useFakeTimers()` / `vi.useFakeTimers()`) for anything involving
  `setTimeout`, debouncing, or `Date.now()`.
- For React components: React Testing Library, query by role/text (not implementation
  details like class names), and avoid shallow rendering.

## Running
```
npm test -- path/to/file.test.ts
npx jest path/to/file.test.ts --coverage
npx vitest run path/to/file.test.ts --coverage
```

## Common pitfalls to avoid
- Don't snapshot-test everything — snapshots are cheap to write but weak at catching
  regressions and easy to blindly re-approve. Use for stable, human-reviewable output only.
- Watch for tests that pass because an unawaited promise never resolved during the test.
