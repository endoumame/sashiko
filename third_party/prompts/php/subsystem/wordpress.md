# WordPress Subsystem Guide

## Security
- Input: `sanitize_text_field()`, `sanitize_email()`, `absint()`
- Output: `esc_html()`, `esc_attr()`, `esc_url()`, `esc_js()`
- CSRF: `wp_nonce_field()` / `wp_verify_nonce()` / `check_admin_referer()`
- Auth: `current_user_can()` before every privileged action
- SQL: `$wpdb->prepare()` with %s, %d, %f placeholders - ALWAYS

## Database
- NEVER concatenate variables into `$wpdb->query()` SQL
- Use `$wpdb->prefix` for table names
- `$wpdb->insert()`, `$wpdb->update()` auto-prepare

## REST API
- `permission_callback` is MANDATORY on all endpoints
- `sanitize_callback` for each parameter
- Return `WP_REST_Response` or `WP_Error`, never echo

## Hooks
- `add_action()` / `add_filter()` priority matters (default 10)
- `remove_action()` must match exact callback and priority
- Anonymous closures cannot be reliably removed

## Common Regressions
1. SQL injection: `$wpdb->query()` without `$wpdb->prepare()`
2. XSS: missing `esc_*()` on output
3. Auth bypass: missing `current_user_can()` check
4. REST API open: missing `permission_callback`
5. CSRF: missing nonce verification
