# Database Subsystem Guide

## Connection Management

### PDO
- Always set `PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION`
- Set `PDO::ATTR_EMULATE_PREPARES => false` for true prepared statements
- Set `PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC` to avoid numeric indexes
- Persistent connections (`PDO::ATTR_PERSISTENT`): may share state unexpectedly
- Connection encoding: set `charset=utf8mb4` in DSN for full Unicode support

### MySQLi
- Always use `mysqli_report(MYSQLI_REPORT_ERROR | MYSQLI_REPORT_STRICT)`
- `$mysqli->real_escape_string()` is NOT sufficient alone - use prepared statements
- `$mysqli->set_charset('utf8mb4')` for proper encoding

## Prepared Statements

### Parameterization Rules
- Table and column names CANNOT be parameterized - validate against whitelist
- `LIKE` wildcards (`%`, `_`) in user input need manual escaping
- `IN (?)` clauses: use `str_repeat('?,', count($ids) - 1) . '?'`
- `ORDER BY` direction: validate against `['ASC', 'DESC']`
- `LIMIT` / `OFFSET`: cast to integer

### PDO Prepared Statements
```php
// Named parameters
$stmt = $pdo->prepare("SELECT * FROM users WHERE email = :email");
$stmt->execute(['email' => $email]);

// Positional parameters
$stmt = $pdo->prepare("SELECT * FROM users WHERE id = ? AND status = ?");
$stmt->execute([$id, $status]);
```

### Common Mistakes
- Reusing statement with different parameter count
- Forgetting to bind LOB parameters with `PDO::PARAM_LOB`
- `PDO::PARAM_INT` not enforced for string values in some drivers

## Transactions

### ACID Compliance
- `BEGIN` / `COMMIT` / `ROLLBACK` must be balanced
- Exception in transaction body: MUST rollback
- Nested transactions: check if driver supports savepoints
- Long-running transactions: may cause lock contention

### Transaction Patterns
```php
// Correct pattern with PDO
$pdo->beginTransaction();
try {
    // ... operations ...
    $pdo->commit();
} catch (\Throwable $e) {
    $pdo->rollBack();
    throw $e;
}
```

### Deadlock Handling
- Detect deadlock exceptions (MySQL error 1213)
- Retry with backoff for transient deadlocks
- Minimize transaction scope to reduce deadlock window
- Acquire locks in consistent order across transactions

## Migration Safety

### Schema Changes
- Adding NOT NULL column without default: breaks existing rows
- Dropping column: ensure no code references it
- Renaming column: requires code change simultaneously
- Adding index: may lock large tables - use `ALGORITHM=INPLACE` on MySQL
- Changing column type: verify data compatibility

### Data Migrations
- Batch large updates to avoid memory/lock issues
- Test rollback path before deploying
- Validate data integrity after migration
- Handle NULL values in data transformation

## Query Performance

### N+1 Detection
- Loading relations in loops without eager loading
- Verify `with()` / `JOIN` used for related data
- Use query logging to detect (Laravel: `DB::enableQueryLog()`)

### Index Usage
- WHERE conditions should match available indexes
- Composite index column order matters
- `LIKE '%pattern'` cannot use index (leading wildcard)
- Function on column prevents index usage: `WHERE YEAR(created_at) = 2024`

## Common Database Regressions

1. **SQL injection**: Raw query with user input
2. **Missing transaction**: Multi-step operations without atomicity
3. **Connection leak**: Unclosed connections in long-running processes
4. **Deadlock**: Inconsistent lock ordering in transactions
5. **N+1 queries**: Lazy loading in loops
6. **Migration failure**: Non-reversible schema change
7. **Encoding**: Wrong charset causing data corruption
8. **Lock timeout**: Long transactions blocking others
