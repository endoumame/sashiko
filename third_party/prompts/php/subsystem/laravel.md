# Laravel Subsystem Guide

## Eloquent ORM
- `$fillable` / `$guarded` MUST be defined to prevent mass assignment
- `hasOne()` / `belongsTo()` can return null - handle nullable access
- `hasMany()` / `belongsToMany()` return empty collections, never null
- `whereRaw()`, `selectRaw()`, `DB::raw()` need parameterization
- `updateOrCreate()` / `firstOrCreate()` are not atomic - race condition risk
- `chunk()` / `lazy()` for memory-efficient large result processing

## Security
- `{{ }}` auto-escapes; `{!! !!}` is raw - audit every raw usage
- `$request->validated()` returns only validated data - safe for assignment
- `$request->all()` returns ALL input - unsafe for mass assignment
- Web routes have CSRF middleware by default; API routes use token auth
- Route model binding can leak data across tenants

## Queues
- Jobs may execute multiple times - ensure idempotency
- `dispatch()->afterCommit()` ensures DB state is committed first
- `ShouldBeUnique` prevents duplicate dispatch

## Common Regressions
1. N+1 queries: missing `with()` eager loading
2. Mass assignment: `$request->all()` or missing `$fillable`
3. Auth bypass: missing middleware on routes
4. Raw SQL injection: `whereRaw()` / `DB::raw()` without bindings
5. Null on relation: nullable hasOne/belongsTo access
