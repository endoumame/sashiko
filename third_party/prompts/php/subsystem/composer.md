# Composer & Autoloading Subsystem Guide

## PSR-4 Autoloading

### Namespace Rules
- Namespace MUST match directory structure exactly
- Case sensitivity: `App\Services\UserService` -> `app/Services/UserService.php`
  - Note: some filesystems are case-insensitive but autoloader is case-sensitive
- Root namespace defined in `composer.json` `autoload.psr-4`
- Dev autoload (`autoload-dev.psr-4`) for tests only

### Common Mistakes
- Namespace doesn't match file path
- Missing trailing backslash in composer.json namespace mapping
- Class name doesn't match filename
- Using `require` / `include` instead of autoloading

## Dependency Management

### composer.json
- `require`: Production dependencies
- `require-dev`: Development-only (tests, debugging tools)
- Version constraints: `^` (compatible), `~` (approximate), exact
- `composer.lock`: MUST be committed for applications, NOT for libraries
- `minimum-stability`: affects entire dependency tree

### Security
- `composer audit`: Check for known vulnerabilities
- Pin exact versions for production deployments
- Verify package authenticity (packagist vs custom repositories)
- Don't run `composer install` as root

## PSR Standards Compliance

### PSR-12 Coding Style
- `declare(strict_types=1)` at top of file
- One class per file
- Opening brace for classes on next line
- Opening brace for methods on next line
- Opening brace for control structures on same line
- Use blocks must be sorted and grouped

### PSR-7 HTTP Messages
- Messages are immutable - `with*()` methods return new instance
- Don't modify request/response objects in place
- Stream body: read once, rewind if needed

### PSR-11 Container
- `has()` before `get()` or handle `NotFoundException`
- Don't use container as service locator (anti-pattern)
- Prefer constructor injection

### PSR-3 Logging
- Log levels: emergency, alert, critical, error, warning, notice, info, debug
- Context array for structured data: `$logger->error('msg', ['id' => $id])`
- Never log sensitive data (passwords, tokens, PII)

## Package Scripts

### Composer Scripts
- `pre-install-cmd` / `post-install-cmd`: Hook into install
- `pre-update-cmd` / `post-update-cmd`: Hook into update
- Custom scripts in `scripts` section
- Beware: scripts run with composer's PHP version

## Common Composer Regressions

1. **Autoload miss**: Namespace doesn't match directory
2. **Dev in production**: `require-dev` package used in production code
3. **Version conflict**: Incompatible dependency constraints
4. **Missing lock file**: Inconsistent installs across environments
5. **Post-install script**: Fails on clean install (missing dependencies)
