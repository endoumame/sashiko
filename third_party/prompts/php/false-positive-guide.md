# False Positive Prevention for PHP Reviews

## Core Rule
**Never report a bug you cannot prove with a concrete code path.**

## Framework Protection Awareness
Before reporting an issue, check if the framework already handles it:
- Eloquent `where()` parameterizes automatically - NOT SQL injection
- Blade `{{ }}` auto-escapes with htmlspecialchars() - NOT XSS
- Laravel CSRF middleware is on by default for web routes
- Symfony Twig auto-escapes by default
- WordPress `esc_html()`, `esc_attr()` handle output escaping

## Type Declaration Trust
- If a parameter has `Type $param` (non-nullable), PHP enforces non-null at runtime
- With `declare(strict_types=1)`, PHP enforces exact types
- Service container / autowiring guarantees typed injection

## Common False Positives to Avoid
1. Reporting SQL injection on parameterized ORM queries
2. Reporting XSS on auto-escaped template output
3. Reporting null reference when type declaration prevents null
4. Reporting missing CSRF on API routes (token auth, not session)
5. Reporting missing validation when FormRequest handles it before controller
6. Reporting foreach on empty array (it simply does nothing)
7. Flagging `Model::findOrFail()` return as nullable (it throws on null)

## Verification Checklist
For each potential issue:
1. Can I show the exact code path that triggers this?
2. Have I verified the path is actually reachable?
3. Does the framework/type system prevent this?
4. Is this production code, not test/debug?
5. Does this apply to the project's PHP version?

If any answer is NO or UNCERTAIN, do not report the issue.
