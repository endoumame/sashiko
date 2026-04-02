# WordPress Subsystem Guide

## Hook System

### Actions & Filters
- `add_action()` / `add_filter()`: priority parameter affects execution order
- Default priority is 10; lower numbers run first
- `remove_action()` / `remove_filter()` must match exact priority and callback
- Anonymous closures cannot be reliably removed
- Late hooks may miss execution if registered after `do_action()` was called
- `did_action()` returns count of times action has fired

### Critical Hook Points
- `init`: Safe for most registrations (post types, taxonomies)
- `wp_loaded`: After all plugins loaded
- `admin_init`: Admin-only initialization
- `wp_enqueue_scripts`: Front-end scripts/styles (NOT `init`)
- `admin_enqueue_scripts`: Admin scripts/styles
- `rest_api_init`: REST API route registration
- `plugins_loaded`: Earliest safe hook for cross-plugin interaction
- `after_setup_theme`: Theme feature support

## Security

### Data Sanitization (Input)
- `sanitize_text_field()`: Strips tags, removes extra whitespace
- `sanitize_email()`: Removes invalid email characters
- `sanitize_file_name()`: Removes special characters
- `absint()`: Absolute integer
- `wp_kses()` / `wp_kses_post()`: Allow specific HTML
- ALWAYS sanitize on input, NEVER trust `$_GET`, `$_POST`, `$_REQUEST`

### Data Escaping (Output)
- `esc_html()`: For HTML content
- `esc_attr()`: For HTML attributes
- `esc_url()`: For URLs (href, src)
- `esc_js()`: For inline JavaScript
- `esc_textarea()`: For textarea content
- `wp_kses()`: For allowing specific HTML tags
- ALWAYS escape on output - even internal data

### Nonce Verification
- `wp_create_nonce()` / `wp_nonce_field()`: Generate CSRF tokens
- `wp_verify_nonce()`: Verify in handlers
- `check_admin_referer()`: Admin form verification
- `check_ajax_referer()`: AJAX request verification
- **CRITICAL**: Every form submission and AJAX handler MUST verify nonce

### Capability Checks
- `current_user_can()`: Check before performing action
- `manage_options`: Admin-level capability
- `edit_posts`: Editor-level capability
- Custom capabilities: verify they are registered and assigned
- **CRITICAL**: Every admin action MUST check capabilities

## Database

### $wpdb Usage
- `$wpdb->prepare()`: ALWAYS use for user input
  - Uses `%s` (string), `%d` (integer), `%f` (float) placeholders
  - NEVER concatenate variables into SQL
- `$wpdb->insert()`, `$wpdb->update()`: Auto-prepare values
- `$wpdb->query()` with raw SQL: MUST use `$wpdb->prepare()` first
- Table prefix: Always use `$wpdb->prefix` or `$wpdb->posts`, etc.
- `$wpdb->last_error`: Check after operations

**Example (BAD)**:
```php
$wpdb->query("SELECT * FROM wp_users WHERE ID = " . $_GET['id']);
```

**Example (GOOD)**:
```php
$wpdb->get_row($wpdb->prepare(
    "SELECT * FROM {$wpdb->users} WHERE ID = %d",
    intval($_GET['id'])
));
```

### Custom Tables
- Use `dbDelta()` for table creation/modification in activation hook
- Always prefix table names with `$wpdb->prefix`
- Use proper column types and indexes

## REST API

### Endpoint Registration
- `register_rest_route()`: Define in `rest_api_init` hook
- `permission_callback`: **MANDATORY** - controls access
  - `__return_true` for public endpoints only
  - Check capabilities for protected endpoints
- `sanitize_callback`: Validate/sanitize each parameter
- `validate_callback`: Type and format validation

### Response Handling
- `WP_REST_Response`: Proper response with status code
- `WP_Error`: Proper error response
- Never echo/die in REST callbacks - return response objects

## Plugin/Theme Development

### Activation & Deactivation
- `register_activation_hook()`: Run on plugin activation
- `register_deactivation_hook()`: Clean up on deactivation
- Flush rewrite rules on activation if registering post types
- Create database tables in activation hook

### Options API
- `get_option()` / `update_option()` / `delete_option()`
- `add_option()`: Only adds if option doesn't exist
- Autoloaded options: keep small data autoloaded, large data not
- `wp_options` table: avoid storing large serialized data

### Transients
- `set_transient()` / `get_transient()`: Cache with expiration
- `delete_transient()`: Remove before data changes
- External object cache may handle transients differently
- Never rely on transient existence for functionality

## File Operations

### Uploads
- Use `wp_handle_upload()` for user uploads
- `wp_upload_dir()` for upload directory path
- Verify file type with `wp_check_filetype()`
- Set proper permissions on uploaded files
- Never store uploads in plugin directory

### Filesystem API
- Use `WP_Filesystem` API for file operations
- Direct file operations may fail on restricted hosts
- Request filesystem credentials if needed

## Common WordPress Regressions

1. **SQL injection**: Using `$wpdb->query()` without `$wpdb->prepare()`
2. **XSS**: Missing `esc_html()` / `esc_attr()` on output
3. **CSRF**: Missing nonce verification in form handlers
4. **Auth bypass**: Missing `current_user_can()` check
5. **REST API open**: Missing `permission_callback` on endpoints
6. **Unescaped output**: Using `echo` instead of proper escaping
7. **Direct file access**: Missing `ABSPATH` check at top of files
8. **Option injection**: Unsanitized data in `update_option()`
9. **Hook priority**: Wrong priority causing unexpected execution order
10. **Global state**: Modifying `$post` or `$wp_query` without restoring
