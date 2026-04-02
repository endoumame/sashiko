# PHP Review Prompts

AI-assisted code review prompts for PHP projects (Laravel, Symfony, WordPress, and generic PHP).
Works with Claude Code and other AI tools.

## Quick Start

```bash
cd scripts
./claude-setup.sh
```

This installs:
- PHP skill that auto-loads in PHP project directories
- Slash commands for review, debug, and verify workflows

## Available Commands

| Command | Description |
|---------|-------------|
| `/php-review` | Review PHP commits for regressions |
| `/php-debug` | Debug PHP errors, crashes, and issues |
| `/php-verify` | Verify findings against false positive patterns |

## Structure

```
php/
├── review-core.md              # Main review protocol (9 stages)
├── technical-patterns.md       # PHP-specific patterns and rules
├── common-bugs.md              # Common PHP bug patterns
├── false-positive-guide.md     # False positive elimination guide
├── inline-template.md          # Review report template
├── debugging.md                # Debugging protocol
├── README.md                   # This file
├── skills/
│   └── php.md                  # Skill definition (auto-loads context)
├── slash-commands/
│   ├── php-review.md           # /php-review command
│   ├── php-debug.md            # /php-debug command
│   └── php-verify.md           # /php-verify command
├── scripts/
│   └── claude-setup.sh         # Installation script
├── patterns/                   # Bug pattern documentation (extensible)
└── subsystem/
    ├── subsystem.md            # Trigger rules for subsystem loading
    ├── laravel.md              # Laravel framework guide
    ├── symfony.md              # Symfony framework guide
    ├── wordpress.md            # WordPress guide
    ├── database.md             # Database and SQL patterns
    ├── composer.md             # Composer and autoloading
    ├── testing.md              # PHPUnit, Pest testing patterns
    ├── api.md                  # REST API patterns
    ├── queue.md                # Queue and job processing
    ├── auth.md                 # Authentication and authorization
    └── caching.md              # Caching and session management
```

## Review Protocol Overview

The PHP review protocol adapts Sashiko's multi-stage approach:

1. **Stage 1**: Analyze commit main goal (architecture, breaking changes)
2. **Stage 2**: High-level implementation verification
3. **Stage 3**: Execution flow verification (null refs, type errors, exceptions)
4. **Stage 4**: Resource management (DB connections, file handles, transactions)
5. **Stage 5**: Concurrency and state management (sessions, cache, locking)
6. **Stage 6**: Security audit (SQL injection, XSS, CSRF, command injection, etc.)
7. **Stage 7**: Framework-specific review (Laravel/Symfony/WordPress patterns)
8. **Stage 8**: Verification and severity estimation (dedup, false positive elimination)
9. **Stage 9**: Report generation (inline-template.md format)

## Supported Frameworks

### Laravel
- Eloquent ORM patterns (relations, mass assignment, query builder)
- Blade template security (escaping, raw output)
- Middleware and routing security
- Queue and job processing
- Service container and dependency injection

### Symfony
- Doctrine ORM patterns (entities, transactions, flush)
- Twig template security
- Security component (firewalls, voters, authentication)
- Messenger (queue) patterns
- Service container and autowiring

### WordPress
- Hook system (actions, filters, priorities)
- Data sanitization and escaping functions
- Nonce verification and capability checks
- $wpdb prepared statements
- REST API endpoint security

### Generic PHP
- PSR compliance (PSR-4, PSR-7, PSR-12)
- Type system and strict types
- Error handling and exception hierarchy
- Resource management
- Security best practices

## Semcode Integration

These prompts work best with [semcode](https://github.com/facebookexperimental/semcode)
for fast code navigation and semantic search.

## License

See [LICENSE](../LICENSE) for license information.
