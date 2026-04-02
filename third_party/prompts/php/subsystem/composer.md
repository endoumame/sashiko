# Composer & Autoloading Guide

## PSR-4 Autoloading
- Namespace MUST match directory structure (case-sensitive)
- Root namespace defined in composer.json autoload.psr-4
- Trailing backslash required in namespace mapping
- Class name must match filename exactly

## Common Issues
- Namespace doesn't match file path
- Dev dependency used in production code (require-dev vs require)
- Missing `composer dump-autoload` after path changes
- Case sensitivity mismatches (works on macOS, fails on Linux)
