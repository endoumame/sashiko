# Call Chain Analysis for PHP

## Tracing Method Call Chains

When analyzing PHP code changes, trace call chains to understand the full impact:

1. **Controller -> Service -> Repository**: Follow the request lifecycle
2. **Middleware pipeline**: Understand what validation/auth happens before your code
3. **Event listeners/subscribers**: Check what side-effects are triggered
4. **Model events**: creating/created/updating/updated observers
5. **Queue dispatch chains**: Job -> handler -> nested dispatches

## Proving Bugs Are Real

Before reporting a bug, you MUST prove it can actually be triggered:
1. Find a concrete call path from an entry point (route, command, job) to the bug site
2. Show that the preconditions for the bug are satisfiable
3. Verify that no middleware, form request, or type declaration prevents the bad state

## PHP-Specific Call Patterns

- **Late static binding**: `static::method()` may resolve to a child class
- **Magic methods**: `__get`, `__set`, `__call` intercept property/method access
- **Closures**: Check `$this` binding context and `use` variable capture
- **Traits**: Method resolution order matters when traits conflict
- **Constructor promotion**: `public function __construct(private Type $prop)` - verify visibility
