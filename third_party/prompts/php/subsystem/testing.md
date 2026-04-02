# Testing Patterns Guide

## PHPUnit
- `assertEquals()` uses loose comparison; `assertSame()` is strict
- Use `assertSame()` for security-sensitive assertions
- Data providers must be static (PHPUnit 10+)
- `@covers` / `#[CoversClass]` limits coverage attribution

## Common Testing Issues
- False positive tests: test passes but doesn't verify the right thing
- Test pollution: shared state between tests (missing tearDown)
- Mocking too much: mocks hide actual bugs
- Time-dependent tests: fail at specific times/dates
- Database state: tests depend on execution order
