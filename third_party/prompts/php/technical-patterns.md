# PHP Technical Deep-dive Patterns

## Core Instructions

- Trace full execution flow, gather additional context from the call chain
- IMPORTANT: never make assumptions based on return types, PHPDoc, or comments -
  explicitly verify the code is correct by tracing concrete execution paths
- IMPORTANT: never skip any steps just because you found a bug in a previous step
- IMPORTANT: PHP documentation and comments are sometimes incomplete or outdated.
  When relying on documentation or comments:
  - Always read the ACTUAL IMPLEMENTATION, not just the docblock
  - Check for conditional behavior based on PHP version or extensions
  - If a method docblock says "returns X" but the code has conditional paths,
    verify which path applies
- Never report errors without checking to see if the error is impossible in the
  call path you found
  - Some call paths might always validate input before reaching the function
  - Do not recommend defensive programming unless it fixes a proven bug

## Type System

### Type Declarations
- PHP 8.x union types: `int|string`, `Foo|Bar`
- PHP 8.1+ intersection types: `Foo&Bar`
- PHP 8.1+ enum types: back values, methods, interfaces
- PHP 8.2+ DNF types: `(Foo&Bar)|null`
- Nullable shorthand: `?string` equals `string|null`
- `void` return type means no return value (not even null)
- `never` return type (PHP 8.1+) means function never returns normally
- `mixed` accepts any type including null
- `static` return type for fluent interfaces

### Type Coercion Pitfalls
- `==` performs type juggling: `"0" == false` is `true`, `"" == false` is `true`
- `===` is strict comparison - ALWAYS prefer for security-sensitive checks
- `in_array()` without `true` as third parameter uses loose comparison
- `array_search()` without `true` as third parameter uses loose comparison
- `switch` uses loose comparison; `match` (PHP 8.0+) uses strict
- `intval("0x1A")` returns 0, not 26 (use explicit base parameter)
- `(int) "123abc"` returns 123 silently (no warning in PHP 8.0)
- `json_decode()` returns `null` on failure AND for valid JSON `"null"`
- Empty string `""` is falsy, but `"0"` is also falsy
- Empty array `[]` is falsy

### Null Safety
- Nullsafe operator `?->` (PHP 8.0+): `$obj?->method()` returns null if $obj is null
- Null coalescing `??`: `$a ?? $b` returns $a if not null
- Null coalescing assignment `??=`: `$a ??= $b`
- `isset()` returns false for null values AND undefined variables
- `array_key_exists()` returns true even if value is null
- `is_null()` triggers notice on undefined, `isset()` does not

## Error Handling

### Exception Hierarchy
```
Throwable
├── Error (engine errors - generally should NOT be caught)
│   ├── TypeError
│   ├── ValueError (PHP 8.0+)
│   ├── ArithmeticError
│   │   └── DivisionByZeroError
│   ├── ParseError
│   └── UnhandledMatchError (PHP 8.0+)
└── Exception
    ├── RuntimeException
    ├── LogicException
    ├── InvalidArgumentException
    ├── PDOException
    └── ... (framework-specific)
```

### Exception Best Practices
- Catch specific exceptions, not generic `\Exception` or `\Throwable`
  - Exception: top-level error handlers and middleware
- Always re-throw or log caught exceptions; never silently swallow
- Use `previous` exception parameter for exception chaining
- Don't use exceptions for control flow
- Framework exceptions should extend framework base exception

### Error Suppression
- NEVER use `@` operator to suppress errors in new code
- `@` hides errors including fatal ones - use proper error handling
- Some legacy code uses `@` for functions that emit warnings (e.g., `@fopen`)
  - Acceptable only if return value is checked immediately after

### Return Values vs Exceptions
- Functions returning `false` on failure (legacy PHP API): ALWAYS check return
- Functions that can return `false|Type`: use `=== false` not `!$result`
  - `strpos()` returns `0` for match at position 0 - `!strpos()` is WRONG
  - `array_search()` returns `0` for match at index 0

## Security Patterns

### SQL Injection Prevention
- ALWAYS use parameterized queries / prepared statements
- NEVER concatenate user input into SQL strings
- Even with ORM: verify raw queries, `whereRaw()`, `DB::raw()`, `selectRaw()`
- Table/column names cannot be parameterized - validate against whitelist
- `LIKE` patterns need manual escaping of `%` and `_`

### XSS Prevention
- HTML output: use `htmlspecialchars($str, ENT_QUOTES, 'UTF-8')`
- Template engines: verify auto-escaping is enabled
  - Blade: `{{ }}` escapes, `{!! !!}` does NOT
  - Twig: auto-escapes by default, `|raw` disables
- JSON in HTML: use `json_encode($data, JSON_HEX_TAG | JSON_HEX_AMP)`
- URLs: use `urlencode()` for query parameters
- Never trust `$_SERVER['HTTP_HOST']` or `$_SERVER['REQUEST_URI']` for output

### Command Injection Prevention
- NEVER use `exec()`, `system()`, `shell_exec()`, `passthru()`, `proc_open()`,
  backticks with unsanitized input
- Use `escapeshellarg()` for arguments, `escapeshellcmd()` for commands
- Prefer PHP native functions over shell commands when possible

### Deserialization Safety
- NEVER use `unserialize()` with untrusted data
- Use `json_decode()` / `json_encode()` instead
- If unserialize is required, use `allowed_classes` option (PHP 7.0+):
  `unserialize($data, ['allowed_classes' => [Foo::class]])`

### File Upload Safety
- Validate MIME type server-side (don't trust client Content-Type)
- Generate random filenames, never use user-supplied names directly
- Store uploads outside web root
- Check file size limits
- Use `move_uploaded_file()`, never `copy()` or `rename()`
- Verify `is_uploaded_file()` before processing

### Session Security
- Use `session_regenerate_id(true)` after login
- Set proper cookie parameters: httponly, secure, samesite
- Validate session data integrity
- Implement proper session timeout

### CSRF Protection
- All state-changing requests must include CSRF token
- Verify token on POST/PUT/PATCH/DELETE requests
- Token must be tied to user session
- Framework-specific: verify middleware is not bypassed

## Resource Management

### Database Connections
- PDO connections should use persistent connections carefully
- Always close cursors/statements when done with large result sets
- Transaction blocks must have both commit AND rollback paths
- Use try/finally for transaction safety:
```php
$pdo->beginTransaction();
try {
    // operations
    $pdo->commit();
} catch (\Throwable $e) {
    $pdo->rollBack();
    throw $e;
}
```

### File Handling
- Always close file handles (use `try/finally` or let scope handle it)
- Check `fopen()` return value before using handle
- Use `file_get_contents()` / `file_put_contents()` for simple operations
- For large files, use streaming (`fread` in chunks)
- Lock files when concurrent access is possible (`flock()`)

### Memory Management
- PHP has automatic garbage collection, but watch for:
  - Circular references (use `WeakReference` or `WeakMap` in PHP 8.0+)
  - Large arrays in loops (unset when no longer needed)
  - Generators for large datasets instead of loading all into memory
  - `yield` for iterating over large result sets

## Common Anti-Patterns

### Array Access
- `$arr['key']` on undefined key: notice in PHP 7, warning in PHP 8
- Use `$arr['key'] ?? $default` or `array_key_exists()`
- `count()` on non-countable: TypeError in PHP 8 (was warning in 7)
- `array_merge()` in loops is O(n^2) - use spread operator or `array_push()`

### String Operations
- `strlen()` counts bytes, not characters - use `mb_strlen()` for UTF-8
- `substr()` counts bytes - use `mb_substr()` for UTF-8
- `strtolower()` / `strtoupper()` are not Unicode-aware - use `mb_` variants
- Regex with Unicode: use `u` modifier (`/pattern/u`)
- `str_contains()`, `str_starts_with()`, `str_ends_with()` (PHP 8.0+)
  are preferred over `strpos()` hacks

### Class / Object Patterns
- Constructor promotion (PHP 8.0+): verify visibility is intentional
- Readonly properties (PHP 8.1+): cannot be modified after initialization
- Enums (PHP 8.1+): cannot be instantiated with `new`
- `clone` is shallow by default - implement `__clone()` for deep copy
- Late static binding: `static::` vs `self::` matters for inheritance
- Abstract classes cannot be instantiated
- Interface methods must be public

### Autoloading
- PSR-4 compliance: namespace must match directory structure
- Verify `composer.json` autoload section matches actual paths
- Case sensitivity: filesystem may be case-insensitive but autoloader is strict

## PHP Version Compatibility

### Breaking Changes to Watch For
- PHP 8.0: Named arguments - parameter name changes break callers
- PHP 8.0: `match` is a reserved word
- PHP 8.0: String-number comparison changes (`0 == "foo"` now false)
- PHP 8.1: Fibers, enums, readonly properties, intersection types
- PHP 8.1: Return type of internal methods (stricter)
- PHP 8.2: Readonly classes, dynamic properties deprecated
- PHP 8.2: `${var}` string interpolation deprecated
- PHP 8.3: Typed class constants, `json_validate()`, `#[\Override]`
- PHP 8.4: Property hooks, asymmetric visibility

### Deprecation Patterns
- Always check if deprecated features are being newly introduced
- `utf8_encode()` / `utf8_decode()` removed in PHP 8.2
- Dynamic properties deprecated in PHP 8.2 (use `#[\AllowDynamicProperties]`)
- `${}` string interpolation deprecated in PHP 8.2

## Testing Patterns

- Test code has different standards:
  - Mocking database calls is expected
  - Test assertions may use loose comparison intentionally
  - Test fixtures may contain intentionally invalid data
- BUT: test code should still be free of security issues
- Verify test actually tests what it claims (check assertion targets)

## Performance Considerations

- Don't flag performance unless it causes measurable regression:
  - N+1 query patterns in loops (check if eager loading exists)
  - Loading entire tables into memory
  - Regex in tight loops when string functions suffice
  - Synchronous external HTTP calls in request lifecycle
