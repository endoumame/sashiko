# API Subsystem Guide

## REST API Design

### Request Validation
- Validate ALL input parameters (query, body, headers)
- Type-check numeric IDs, UUIDs, enums
- Validate content-type header matches expected format
- Limit request body size
- Validate pagination parameters (page, limit with max)

### Response Format
- Consistent JSON structure across endpoints
- Include appropriate HTTP status codes
- Don't leak internal details in error responses
  - Stack traces, SQL queries, file paths in production
- Use `JsonResponse` or equivalent, not raw `echo json_encode()`

### Authentication
- API token in `Authorization: Bearer` header, not query string
- Token validation on every request (not cached indefinitely)
- Rate limiting per user/IP
- Token expiration and refresh mechanism

### Authorization
- Resource-level permission checks (not just authentication)
- Verify tenant isolation in multi-tenant APIs
- `404` for resources that exist but user can't access (don't leak existence)

## Input Handling

### JSON Body
- `json_decode()` returns null on failure - check with `json_last_error()`
- Use `json_decode($body, true, 512, JSON_THROW_ON_ERROR)` for safety
- Validate JSON schema before processing
- Handle malformed JSON gracefully

### File Uploads
- Validate MIME type server-side
- Check file size limits
- Generate unique filenames
- Store outside web root
- Scan for malware if possible

## Output Safety

### Sensitive Data
- Never include passwords, tokens, or secrets in responses
- Filter sensitive fields from model serialization
- Use dedicated response DTOs/transformers
- Log sanitized request data (strip auth headers)

### Pagination
- Cursor-based for large/real-time datasets
- Offset-based for traditional pagination
- Include total count only when needed (expensive on large tables)
- Cap maximum page size

## Common API Regressions

1. **Auth bypass**: Missing authentication middleware on new endpoint
2. **Data leak**: Serializing full model with sensitive fields
3. **Injection**: Unvalidated input in database queries
4. **DoS**: No rate limiting or pagination limits
5. **Broken pagination**: Off-by-one in offset calculation
6. **Version break**: Changing response format without versioning
