# Python

## Framework
Default to **pytest** unless the project clearly uses `unittest` already (look for
`unittest.TestCase` subclasses or a `test*.py` pattern with `self.assert...` calls — in that
case, match it rather than mixing frameworks in one repo).

## File placement & naming
- `test_<module>.py` or `<module>_test.py`, mirroring whatever the repo already does.
- Usually lives in a top-level `tests/` directory mirroring `src/` structure, or alongside
  the module — check the existing layout first.

## Style
- Plain `assert` statements (pytest rewrites these with useful diffs) — don't use
  `self.assertEqual` unless matching an existing `unittest`-style suite.
- Use `pytest.raises(SomeError)` as a context manager for exception tests.
- Use `@pytest.mark.parametrize` for table-driven / boundary-value cases instead of copy-pasted
  near-identical test functions.
- Fixtures (`@pytest.fixture`) for shared setup instead of `setUp`/`tearDown`, unless matching
  an existing `unittest` suite.

## Mocking
- `unittest.mock.patch` / `MagicMock`, or the `pytest-mock` `mocker` fixture if it's already a
  dependency.
- Mock at the boundary the code actually calls (e.g. patch `requests.get` where it's used, not
  where it's defined, following the standard "patch where imported" pytest-mock rule).
- For time-dependent code, patch `time.time`/`datetime.now` or use `freezegun` if present in
  the project's dependencies.

## Running
```
pytest path/to/test_file.py -v
pytest --cov=src --cov-report=term-missing   # if pytest-cov is installed
```

## Common pitfalls to avoid
- Don't test private (`_leading_underscore`) helpers directly if a public method exercises
  them adequately — test through the public interface first.
- Watch for mutable default arguments and shared fixture state causing cross-test pollution.
