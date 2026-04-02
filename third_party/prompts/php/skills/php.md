---
name: php
description: Load anytime the working directory is a PHP project. PHP-specific knowledge, framework patterns, code review, and debugging protocols. Read this anytime you're in a PHP project tree.
invocation_policy: automatic
---

## ALWAYS READ
1. Load `{{PHP_REVIEW_PROMPTS_DIR}}/technical-patterns.md`

These files are MANDATORY. This skill exists as a framework for loading
additional PHP prompts.

## Configuration

The review prompts directory is configured during installation:
- **PHP_REVIEW_PROMPTS_DIR**: {{PHP_REVIEW_PROMPTS_DIR}}

This variable is set by the installation script when the skill is installed.

## Capabilities

### Patch Review
When asked to review a PHP patch, commit, or series of commits:
1. Load `{{PHP_REVIEW_PROMPTS_DIR}}/review-core.md`
2. Follow the complete review protocol defined there
3. Load subsystem-specific files as directed by review-core.md

### Debugging
When asked to debug a PHP crash, error, or unexpected behavior:
1. Load `{{PHP_REVIEW_PROMPTS_DIR}}/debugging.md`
2. Follow the complete debugging protocol defined there
3. Use error logs, stack traces, and exception output as entry points

### Subsystem Context
When working on PHP code in specific frameworks or subsystems, load the
appropriate context files from `{{PHP_REVIEW_PROMPTS_DIR}}/`:

1. Always read `technical-patterns.md` before loading subsystem specific files

2. Select subsystem specific files as needed:

| Subsystem | Trigger | File |
|-----------|---------|------|
| Laravel | `app/`, Eloquent, Blade, Artisan, `routes/` | `subsystem/laravel.md` |
| Symfony | `Symfony\Component\`, Doctrine, Twig, `services.yaml` | `subsystem/symfony.md` |
| WordPress | `wp_`, `add_action`, `$wpdb`, `wp-content/` | `subsystem/wordpress.md` |
| Database | PDO, MySQLi, migration, schema | `subsystem/database.md` |
| Composer | `composer.json`, PSR-4, autoload | `subsystem/composer.md` |
| Testing | PHPUnit, Pest, `@test`, `tests/` | `subsystem/testing.md` |
| API | REST, JSON response, endpoint | `subsystem/api.md` |
| Queue | `ShouldQueue`, dispatch, job, worker | `subsystem/queue.md` |
| Auth | middleware, guard, policy, gate, voter | `subsystem/auth.md` |
| Caching | Cache, Redis, Memcached, session | `subsystem/caching.md` |

## Semcode Integration

When available, use semcode MCP tools for efficient code navigation:
- `find_function` / `find_type`: Get function and type definitions
- `find_callchain`: Trace call relationships up and down
- `find_callers` / `find_calls`: Explore call graphs
- `grep_functions`: Search function bodies with regex
- `diff_functions`: Identify changed functions in patches
- `find_commit` / `vcommit_similar_commits`: Search commit history

## Output

- Patch reviews produce `review-inline.txt` when regressions are found
- Debug sessions produce `debug-report.txt` with analysis results
- Both outputs are plain text, suitable for GitHub PRs or code review tools

## Key PHP Conventions

### Error Handling
- Use exceptions for exceptional cases, not control flow
- Catch specific exceptions, not generic `\Throwable`
- Use `declare(strict_types=1)` in all new files

### Security
- Parameterized queries always (never concatenate SQL)
- Escape output always (`htmlspecialchars`, template engine escaping)
- Validate and sanitize all user input
- Use `===` for security-sensitive comparisons
- Use `hash_equals()` for timing-safe token comparison

### Type Safety
- Use type declarations on parameters, returns, and properties
- Prefer strict types (`declare(strict_types=1)`)
- Use union types and nullable types appropriately
- Avoid loose comparisons (`==`) in new code

### Resource Management
- Always handle transaction commit/rollback on all paths
- Close file handles in finally blocks
- Use generators for large dataset iteration
