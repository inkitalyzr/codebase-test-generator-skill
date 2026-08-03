# General fallback (any language/framework not covered by a dedicated reference)

When the language isn't Python, JS/TS, Go, or Java/Kotlin (e.g. Rust, Ruby, PHP, C#, Elixir,
Swift):

1. **Detect the existing test tool first** from the package manifest (Cargo.toml, Gemfile,
   composer.json, .csproj, mix.exs, Package.swift) and existing test file patterns. Match it —
   don't introduce a second test framework into a project that already has one.
2. **If nothing exists yet**, use the ecosystem's de facto standard:
   - Rust → built-in `#[test]` / `#[cfg(test)]` modules
   - Ruby → RSpec (or Minitest if the project is Rails-default and hasn't added RSpec)
   - PHP → PHPUnit
   - C# → xUnit (or NUnit/MSTest if already present)
   - Elixir → ExUnit
   - Swift → XCTest
3. **Apply the same universal structure** regardless of language:
   - Happy path → boundary/edge cases → error handling → regression test for recent bug fixes
   - Mock true externalities only (network, DB, filesystem, clock, randomness)
   - Table/data-driven tests for anything with many similar input/output cases
   - Match existing naming and file-placement conventions rather than inventing new ones
4. **Look up the specific syntax via web search or the project's own dependency docs** if
   unsure of exact assertion/mocking API — don't guess at framework-specific method names.
