# Authentication & Authorization Subsystem Guide

## Authentication

### Password Handling
- ALWAYS use `password_hash()` with `PASSWORD_DEFAULT` or `PASSWORD_ARGON2ID`
- NEVER use MD5, SHA1, or SHA256 for passwords
- Use `password_verify()` for comparison (timing-safe)
- `password_needs_rehash()`: upgrade on login when algorithm changes

### Session-Based Auth
- Regenerate session ID after login: `session_regenerate_id(true)`
- Set secure cookie parameters: `httponly`, `secure`, `samesite=Lax`
- Implement session timeout (idle and absolute)
- Destroy session properly on logout: `session_destroy()` + cookie removal

### Token-Based Auth
- Use cryptographically secure token generation: `bin2hex(random_bytes(32))`
- Store hashed tokens in database, compare with `hash_equals()`
- Implement token expiration
- Allow token revocation

### OAuth/JWT
- Validate JWT signature BEFORE trusting claims
- Check `exp`, `iss`, `aud` claims
- Don't store sensitive data in JWT payload (it's base64, not encrypted)
- Token refresh: short-lived access + long-lived refresh

### Multi-Factor Authentication
- TOTP implementation: verify time window handling
- Backup codes: store hashed, mark as used
- Rate limit verification attempts

## Authorization

### Role-Based Access Control (RBAC)
- Check permissions at controller/service level, not just routes
- Verify role hierarchy is correctly implemented
- Cache permission lookups for performance
- Invalidate cache on role/permission changes

### Policy/Voter Pattern
- Each resource type should have a dedicated policy
- Check authorization BEFORE performing action (not after)
- Handle "not found" vs "not authorized" correctly
  - Return 404 for resources user shouldn't know about
  - Return 403 for resources user knows but can't access

### Middleware Guards
- Verify middleware is applied to ALL protected routes
- Check route groups don't accidentally exclude routes
- API vs web guard: different authentication mechanisms
- Guest middleware: redirect authenticated users

## Common Auth Regressions

1. **Broken auth**: Missing middleware on new route
2. **Privilege escalation**: Checking role name instead of permission
3. **IDOR**: Accessing resource by ID without ownership check
4. **Session fixation**: Missing session regeneration on login
5. **Token leak**: Token in URL, logs, or error messages
6. **Timing attack**: Using `==` instead of `hash_equals()` for tokens
7. **Password in logs**: Logging request body including password field
