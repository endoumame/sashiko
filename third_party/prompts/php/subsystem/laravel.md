# Laravel Subsystem Guide

## Eloquent ORM

### Model Patterns
- `$fillable` / `$guarded` must be defined - prevents mass assignment
- `$casts` array ensures type safety on model attributes
- Accessors (get*Attribute) and mutators (set*Attribute) may transform data
- PHP 8+ accessor syntax: `Attribute::make(get: fn() => ...)`
- Soft deletes: `SoftDeletes` trait changes `delete()` behavior
- Global scopes may silently filter queries

### Relationships
- `hasOne()` / `belongsTo()` can return null - always handle nullable
- `hasMany()` / `belongsToMany()` return empty collections, never null
- Eager loading (`with()`) prevents N+1 but may load excessive data
- `withDefault()` on relationships prevents null access
- Lazy loading may be disabled (`Model::preventLazyLoading()`)

### Query Builder
- `where()` with array values automatically parameterizes
- `whereRaw()`, `selectRaw()`, `orderByRaw()` need manual parameterization
- `DB::raw()` content is NOT escaped - always parameterize
- `firstOrFail()` vs `first()`: former throws, latter returns null
- `updateOrCreate()` is not atomic - race condition possible
- `chunk()` / `lazy()` for memory-efficient large result processing

## Routing & Middleware

### Route Security
- API routes should use `auth:sanctum` or `auth:api` middleware
- Web routes need `web` middleware group (includes CSRF, session)
- `withoutMiddleware()` removes protection - verify intentional
- Route model binding: implicit binding can leak data across tenants
- Rate limiting should be applied to auth and API endpoints

### Middleware Order
- Middleware executes in order defined in `$middleware` / `$middlewareGroups`
- `$middlewarePriority` controls execution order across groups
- `terminate()` method runs AFTER response sent
- Exception in middleware may bypass subsequent middleware

## Blade Templates

### Escaping
- `{{ }}` auto-escapes with `htmlspecialchars()`
- `{!! !!}` outputs RAW HTML - audit every usage
- `@json()` directive safely encodes for JavaScript contexts
- `@verbatim` disables Blade processing - raw output
- Custom Blade directives may not escape

### Components
- Component attributes are escaped by default
- `$attributes->merge()` preserves user attributes - check for XSS
- Anonymous components: verify data binding

## Service Container & Providers

### Dependency Injection
- Constructor injection is automatically resolved
- `app()->make()` / `resolve()` may fail at runtime
- Singleton vs transient: check if shared state causes issues
- Deferred providers load only when needed

### Service Providers
- `register()`: only bind things into the container
- `boot()`: safe to use other services, run after all register()
- Don't resolve services in `register()` - they may not be bound yet

## Validation

### Form Requests
- `authorize()` runs BEFORE `rules()` - can be a security gate
- `validated()` returns only validated data - safe for mass assignment
- `$request->all()` returns ALL input - unsafe for mass assignment
- Custom validation rules must handle edge cases (null, empty)

## Queues & Jobs

### Job Safety
- Jobs may execute multiple times (retry) - ensure idempotency
- `$tries`, `$timeout`, `$backoff` control retry behavior
- Failed jobs go to `failed_jobs` table - check for sensitive data
- Database transactions in jobs: verify commit before dispatch
- `dispatch()->afterCommit()` ensures DB state is committed first

## Artisan Commands

### Command Safety
- `$this->argument()` / `$this->option()` are NOT sanitized
- File paths from arguments need validation
- Long-running commands should handle signals gracefully
- Memory leaks in daemon commands (queue worker patterns)

## Events & Listeners

### Event Patterns
- Queued listeners may fail silently
- Event ordering is not guaranteed across listeners
- `ShouldQueue` on listener makes it async - state may change
- Model events fire in specific order: creating, created, etc.
- `withoutEvents()` suppresses all model events - verify intentional

## Common Laravel Regressions

1. **N+1 queries**: Accessing relations in loops without eager loading
2. **Mass assignment**: Using `$request->all()` or missing `$fillable`
3. **Auth bypass**: Missing middleware on routes
4. **Raw SQL injection**: `whereRaw()` / `DB::raw()` without bindings
5. **Null on relation**: Accessing property on nullable hasOne/belongsTo
6. **Race condition**: `updateOrCreate()` / `firstOrCreate()` concurrent calls
7. **Memory exhaustion**: Loading large collections without chunking
8. **Unqueued notification**: Blocking request with email/SMS sending
