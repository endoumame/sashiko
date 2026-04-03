# PHP Technical Deep-dive Patterns

## Core Instructions
- Trace full execution flow, gather additional context from the call chain
- IMPORTANT: never make assumptions based on PHPDoc, comments, or return type declarations alone - explicitly verify the code is correct by tracing concrete execution paths
- IMPORTANT: never skip any steps just because you found a bug in a previous step
- Never report errors without checking if the error is impossible in the call path

## Type System
- `==` performs type juggling; `===` is strict - prefer strict for security
- `in_array()` and `array_search()` need `true` as 3rd param for strict mode
- `switch` uses loose comparison; `match` (PHP 8.0+) uses strict
- Empty values: `""`, `"0"`, `0`, `0.0`, `null`, `false`, `[]` are all falsy
- `isset()` returns false for null AND undefined; `array_key_exists()` is null-safe

## Error Handling
- Catch specific exceptions, not generic \Exception or \Throwable (except top-level handlers)
- Never silently swallow exceptions
- Functions returning false on failure: use `=== false`, not `!$result`
- `@` error suppression operator hides all errors - avoid in new code

## Security Essentials
- ALWAYS parameterize SQL queries (PDO prepared statements, Eloquent bindings)
- ALWAYS escape output (htmlspecialchars, template engine auto-escaping)
- Use `hash_equals()` for timing-safe token comparison, not `==` or `===`
- Use `password_hash()` / `password_verify()` for passwords, never MD5/SHA
- Use `random_bytes()` / `bin2hex()` for tokens, never `rand()` / `mt_rand()`

## Resource Management
- Database transactions: ALWAYS have both commit AND rollback paths
- File handles: close in `finally` blocks
- Use generators (`yield`) for large dataset iteration
- Long-running processes: beware memory leaks from event listeners, static caches

## Framework Patterns
- Laravel: `$fillable` prevents mass assignment; `{{ }}` auto-escapes; middleware handles CSRF
- Symfony: autowiring resolves dependencies; Twig auto-escapes; security voters for auth
- WordPress: `esc_html()` for output; `$wpdb->prepare()` for SQL; nonces for CSRF

## Common Anti-Patterns
- `array_merge()` in loops: O(n^2) - use spread operator or `array_push()`
- `strlen()` for multibyte: use `mb_strlen()` for UTF-8
- Dynamic properties: deprecated in PHP 8.2
- `extract()`: creates variables from array, makes code hard to follow and unsafe
