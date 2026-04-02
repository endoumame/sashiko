# PHP Type Safety Patterns

## Type Declaration Verification
- `declare(strict_types=1)`: Affects the FILE where it's declared, not the callee
- Union types: `int|string` - verify all branches handle each variant
- Nullable: `?Type` equals `Type|null` - verify null handling
- Intersection: `Foo&Bar` - object must implement both
- `mixed` accepts anything including null

## Common Type Bugs

### Loose vs Strict Comparison
```php
// BUG: "0" == false is true, 0 == "foo" is true (PHP < 8.0)
if ($value == false) { ... }
// FIX: use strict comparison
if ($value === false) { ... }
```

### strpos() / array_search() Return Value
```php
// BUG: strpos returns 0 for match at position 0, which is falsy
if (!strpos($haystack, $needle)) { ... }
// FIX: explicit check for false
if (strpos($haystack, $needle) === false) { ... }
// BETTER (PHP 8.0+): use str_contains()
if (!str_contains($haystack, $needle)) { ... }
```

### json_decode() Ambiguity
```php
// BUG: json_decode returns null both on failure AND for valid JSON "null"
$data = json_decode($json);
if ($data === null) { /* is this an error or valid null? */ }
// FIX: use JSON_THROW_ON_ERROR
$data = json_decode($json, true, 512, JSON_THROW_ON_ERROR);
```

### Nullable Relation Access
```php
// BUG: hasOne/belongsTo can return null
$user->profile->avatar;  // Error if profile is null
// FIX: nullsafe operator
$user->profile?->avatar;
```

## Validation Library Patterns
- Form Requests validate before controller runs
- Manual validation: check if validated data is used vs raw input
- Custom rules: verify they handle null/empty gracefully
