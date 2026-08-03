# Java / Kotlin

## Framework detection
Check `pom.xml`/`build.gradle`:
- JUnit 5 (`org.junit.jupiter`) → preferred default for new tests if the project is on it.
- JUnit 4 (`org.junit.Test` w/o jupiter) → match if that's what exists; don't mix 4 and 5
  annotations in the same file.
- Kotlin projects often use **Kotest** or **JUnit5 + MockK** — check before defaulting to
  plain JUnit/Mockito style.

## File placement & naming
- Maven/Gradle standard layout: `src/test/java/...` (or `src/test/kotlin/...`) mirroring the
  package path of `src/main/...`.
- Class name: `<ClassUnderTest>Test.java` / `...Test.kt`.

## Style
- JUnit 5: `@Test`, `@ParameterizedTest` + `@ValueSource`/`@MethodSource` for boundary/table
  cases instead of many near-duplicate `@Test` methods.
- AssertJ (`assertThat(...).isEqualTo(...)`) if already a dependency — more readable failure
  messages than plain JUnit asserts; otherwise use JUnit's built-in assertions.
- Kotlin: prefer Kotest's `should` DSL or `assertEquals` from `kotlin.test` if that's the
  existing convention; use MockK (not Mockito) for Kotlin-native mocking if present.

## Mocking
- Mockito (`@Mock`, `when(...).thenReturn(...)`) for Java; MockK for idiomatic Kotlin.
- Use `@ExtendWith(MockitoExtension.class)` (JUnit5) or `@RunWith(MockitoJUnitRunner.class)`
  (JUnit4) to wire mocks — match whichever the project already uses.
- For Spring projects, prefer `@WebMvcTest`/`@DataJpaTest` slice tests over booting the full
  `@SpringBootTest` context unless integration-level coverage is specifically wanted.

## Running
```
mvn test -Dtest=ClassNameTest
gradle test --tests "com.example.ClassNameTest"
```

## Common pitfalls to avoid
- Don't spin up a full Spring context for a unit test that doesn't need it — slow and brittle.
- Watch for `equals()`/`hashCode()` not being overridden on data classes used in `isEqualTo`
  comparisons, causing false passes/failures.
