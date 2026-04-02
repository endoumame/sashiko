# Caching Subsystem Guide

## Cache Safety

### Cache Invalidation
- Cache data that changes rarely, invalidate on mutation
- Use cache tags for group invalidation (not all drivers support tags)
- TTL should match data freshness requirements
- Stale-while-revalidate for high-traffic endpoints

### Cache Key Design
- Include all parameters that affect the cached value
- Namespace keys to avoid collisions across features
- Include version/schema in key for format changes
- Avoid user-controlled input in cache keys (injection risk)

### Race Conditions
- Cache stampede: multiple processes regenerate same expired cache
- Use atomic locks: `Cache::lock('key')->get(fn() => ...)`
- `Cache::remember()` with TTL for automatic regeneration
- `Cache::forever()`: explicit invalidation required

## Session Management

### Session Security
- Regenerate ID on authentication state change
- Set `cookie_httponly`, `cookie_secure`, `cookie_samesite`
- Implement idle timeout and absolute timeout
- Store minimal data in session (not full objects)

### Session Drivers
- File: simple but no sharing across servers
- Database: shared, but adds DB load
- Redis/Memcached: fast, shared, with TTL
- Cookie: client-side, limited size, integrity via encryption

### Session Data
- Never store sensitive data unencrypted in session
- Validate session data on read (may be tampered or stale)
- Clear session data on logout: `session_destroy()` or `$request->session()->invalidate()`

## Redis-Specific

### Connection
- Use persistent connections for long-running processes
- Handle connection failures gracefully (cache should be optional)
- Set appropriate timeouts (connect, read, write)

### Data Types
- Strings for simple key-value
- Hashes for structured data
- Sets/sorted sets for collections
- Lists for queues (but use proper queue drivers instead)

### Common Issues
- Key collision between applications (use prefix)
- Memory exhaustion: set `maxmemory` and eviction policy
- Serialization: ensure objects can serialize/deserialize correctly
- TTL not set: keys persist forever, memory grows

## Common Caching Regressions

1. **Stale data**: Cache not invalidated after data mutation
2. **Cache stampede**: Many processes regenerating same key
3. **Session fixation**: Missing session ID regeneration
4. **Key collision**: Same key used for different data
5. **Unbounded growth**: Missing TTL on cache entries
6. **Sensitive data**: Passwords/tokens cached in plaintext
7. **Race condition**: Read-modify-write without locking
