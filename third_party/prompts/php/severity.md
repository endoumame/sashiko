# Severity Classification for PHP Reviews

## Critical
- SQL injection (user input in raw queries)
- Remote code execution (command injection, unsafe deserialization, eval)
- Authentication/authorization bypass
- Data corruption affecting production data

## High
- XSS vulnerabilities (unescaped user output)
- CSRF protection bypass
- Path traversal / local file inclusion
- Mass assignment allowing privilege escalation
- Resource leaks in long-running processes (workers, daemons)
- Unhandled exceptions causing data loss (e.g., uncommitted transactions)
- Type juggling bugs in security-sensitive comparisons

## Medium
- Missing input validation (non-security context)
- N+1 query performance issues
- Incorrect error handling (wrong exception caught, swallowed errors)
- Session handling issues
- Missing null checks on nullable returns
- Incorrect type declarations

## Low
- Code style violations
- Missing PHPDoc
- Suboptimal patterns (not affecting correctness)
- Test code quality issues
