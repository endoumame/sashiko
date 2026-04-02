# Testing Subsystem Guide

## PHPUnit

### Test Structure
- Test classes extend `TestCase` (or framework equivalent)
- Test methods prefixed with `test` or annotated with `@test` / `#[Test]`
- `setUp()` / `tearDown()`: per-test setup/cleanup
- `setUpBeforeClass()` / `tearDownAfterClass()`: per-class (static)

### Assertions
- `assertEquals()`: loose comparison (type juggling)
- `assertSame()`: strict comparison (type + value)
- Use `assertSame()` for security-sensitive checks
- `assertCount()` instead of `assertEquals(3, count($arr))`
- `assertNull()` instead of `assertSame(null, $value)`

### Data Providers
- `@dataProvider` / `#[DataProvider]`: method must be static (PHPUnit 10+)
- Provider returns array of arrays or iterator
- Named datasets help identify failing cases
- Provider runs before `setUp()` - don't depend on test state

### Mocking
- `createMock()`: creates mock with all methods stubbed (return null)
- `createPartialMock()`: only specified methods are stubbed
- `getMockBuilder()`: fine-grained control
- Mock expectations: `expects($this->once())` verifies call count
- Prophecy (if used): different syntax, same concepts

### Coverage
- `@covers` / `#[CoversClass]`: limits coverage attribution
- Verify test actually tests what it claims (not just coverage farming)
- Integration tests may cover code but not validate behavior

## Pest

### Pest-Specific Patterns
- `it()` / `test()`: test definitions
- `expect()`: fluent assertion API
- `beforeEach()` / `afterEach()`: hooks
- `dataset()`: data providers
- Higher-order tests: chained expectations
- `arch()`: architecture testing

## Framework Test Utilities

### Laravel
- `RefreshDatabase` / `DatabaseMigrations` / `DatabaseTransactions`
- `actingAs()`: authenticate as user
- `assertDatabaseHas()` / `assertDatabaseMissing()`
- `Queue::fake()`, `Event::fake()`, `Mail::fake()`: test without side effects
- `$this->get()`, `$this->post()`: HTTP test methods

### Symfony
- `WebTestCase` / `KernelTestCase`
- `$client->request()`: HTTP testing
- `$crawler->filter()`: DOM assertions
- Container access in tests: `self::getContainer()`

## Common Testing Regressions

1. **False positive test**: Test passes but doesn't verify the right thing
2. **Test pollution**: Shared state between tests (missing tearDown)
3. **Mocking too much**: Mock hides actual bugs
4. **Database state**: Tests depend on execution order
5. **Time-dependent**: Tests fail at specific times/dates
