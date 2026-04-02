# False Positive Elimination Guide

## Purpose
This guide helps eliminate false positives before reporting regressions in PHP code.
Apply these checks to every potential issue found.

## Core Principle
**Never report a bug you cannot prove with concrete code paths.**

## Verification Checks

### CHECK 1: Can the Code Path Actually Execute?

Before reporting an issue:

1. **Trace the call path** from entry point to problematic code
2. **Identify all conditions** that must be true to reach it
3. **Verify conditions are possible** in real usage

**False Positive Example**:
```php
/* Reported: Null reference on $user */
function processUser(?User $user): void {
    if ($user === null) {
        return;
    }
    // ... lots of code ...
    $user->getName();  // $user CANNOT be null here!
}
```

### CHECK 2: Are Prerequisites Validated Elsewhere?

PHP frameworks often validate at middleware or form request level, not at
every use site.

**Check**:
- Does middleware validate this input?
- Is this an internal method that assumes valid input from the controller?
- Are there type declarations enforcing the constraint?
- Does the framework guarantee non-null injection?

**False Positive Example**:
```php
/* Internal service - controller validates user exists */
class OrderService {
    public function createOrder(User $user, array $items): Order {
        // No null check needed - type declaration enforces non-null
        return new Order($user->getId(), $items);
    }
}
```

### CHECK 3: Does the Framework Handle This?

Many issues are already handled by the framework:

**False Positive Examples**:
```php
/* Reported: SQL injection in Eloquent query */
User::where('email', $request->input('email'))->first();
// Eloquent parameterizes this automatically - NOT an injection

/* Reported: XSS in Blade template */
{{ $user->name }}
// Blade's {{ }} auto-escapes - NOT an XSS vulnerability
// {!! $user->name !!} WOULD be a real issue

/* Reported: Missing CSRF protection */
// Routes in api.php use token auth, not session - CSRF N/A

/* Reported: No input validation */
// FormRequest class handles validation before controller runs
```

### CHECK 4: Is This Defensive Programming Territory?

Don't recommend checks that can't fail:

**False Positive Example**:
```php
/* Reported: Should check if array is empty before foreach */
foreach ($items as $item) {
    // foreach on empty array simply does nothing - no check needed
}

/* Reported: Should validate return type */
function getUser(): User {
    // Return type declaration enforces this at runtime
    return $this->repository->find($id);
}
```

**Real Issue Example**:
```php
/* This IS an issue - find() can return null */
function getUser(int $id): User {
    return $this->repository->find($id);  // Returns User|null!
}
```

### CHECK 5: Is the Error Handling Actually Wrong?

Some patterns are intentional:

```php
/* Intentional - best-effort cleanup */
@unlink($tempFile);  // Legacy but intentional for temp files

/* Intentional - logging failure should not crash the app */
try {
    $logger->info('action completed');
} catch (\Throwable $e) {
    // Silently continue - logging is non-critical
}
```

### CHECK 6: Is This a Test or Debug Path?

Test code has different standards:

- Mocking may intentionally return unexpected types
- Test data may contain intentionally invalid values
- Debug code may have intentional simplifications
- PHPUnit assertions may use loose comparison by design

### CHECK 7: PHP Version Context

Some issues depend on PHP version:

```php
/* Only an issue on PHP < 8.0 */
if (strpos($haystack, $needle) == false) {
    // On PHP 8.0+ with strict_types, this is still wrong
    // But check the project's minimum PHP version
}

/* Dynamic properties - only deprecated in PHP 8.2+ */
$obj->newProperty = 'value';
// Check composer.json require.php version
```

### CHECK 8: Type Safety with strict_types

Check if `declare(strict_types=1)` is in effect:

```php
declare(strict_types=1);

/* With strict_types, this throws TypeError */
function add(int $a, int $b): int {
    return $a + $b;
}
add("1", "2");  // TypeError in strict mode, works in non-strict

/* Without strict_types, PHP coerces silently */
```

### CHECK 9: Framework-Specific Patterns

**Laravel**:
- `$request->input()` returns `mixed` - but form validation may guarantee type
- `Model::findOrFail()` throws, `Model::find()` returns nullable
- Service container bindings guarantee type at injection
- Middleware may guarantee authentication state
- `abort_if()` / `abort_unless()` terminates execution

**Symfony**:
- Autowiring guarantees typed injection
- ParamConverter resolves or throws before controller
- Security voters handle authorization
- Form validation occurs before handler

**WordPress**:
- Hooks may be called in unexpected order
- Global state (`$wpdb`, `$post`) may be null
- `esc_html()`, `esc_attr()`, etc. handle escaping
- `wp_nonce_field()` / `check_admin_referer()` handle CSRF

## Elimination Process

For each potential issue:

1. [ ] Can I show the exact code path that triggers this?
2. [ ] Have I verified the path is actually reachable?
3. [ ] Is this truly a bug, not defensive programming request?
4. [ ] Have I checked for framework-level validation?
5. [ ] Have I checked for type declaration enforcement?
6. [ ] Is this production code, not test/debug?
7. [ ] Is this actually an issue given the target PHP version?

**If any answer is NO or UNCERTAIN**, do not report the issue.

## Output Format

For issues that pass all checks:

```
VERIFIED ISSUE: [brief description]
Code path: ClassName::method() -> OtherClass::method() -> issue_site
Condition: [what must be true for this to trigger]
Evidence: [code snippet showing the problem]
```

For eliminated false positives:

```
ELIMINATED: [brief description]
Reason: [which check failed and why]
```
