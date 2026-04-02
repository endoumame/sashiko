# Symfony Subsystem Guide

## Dependency Injection

### Service Configuration
- Autowiring resolves type-hinted constructor parameters automatically
- `autoconfigure: true` applies tags based on interfaces implemented
- Public vs private services: private services cannot be fetched from container
- Aliases allow interface-to-implementation mapping
- Compiler passes modify container at build time - verify correctness
- Synthetic services must be set manually - check initialization

### Service Scoping
- Default scope is shared (singleton within request)
- Service subscribers / locators for lazy loading
- Non-shared services create new instances per injection
- Beware shared services with request-dependent state

## Doctrine ORM

### Entity Patterns
- Entity lifecycle callbacks: `@PrePersist`, `@PostUpdate`, etc.
- Lazy loading proxies: accessing unloaded relation triggers query
- Detached entities: `merge()` needed after serialization/deserialization
- Flush behavior: all pending changes written at once
- Cascade operations: `cascade={"persist", "remove"}` affects related entities
- Orphan removal: `orphanRemoval=true` deletes unreferenced children

### Query Patterns
- DQL is parameterized by default with `setParameter()`
- Native queries: use `ResultSetMapping` or parameter binding
- `createQuery()` with string concatenation = SQL injection risk
- Repository custom methods should return typed results
- Pagination: verify total count query for large tables

### Transactions
- `EntityManager::transactional()` handles commit/rollback
- Nested transactions use savepoints
- Exception during flush: EntityManager becomes closed - cannot reuse
- Clear entity manager after batch operations to free memory

## Security

### Authentication
- Authenticators implement `AuthenticatorInterface`
- Firewall configuration order matters - first match wins
- `access_control` rules evaluated top-down, first match applies
- Remember me tokens: verify secure storage and rotation
- Stateless APIs should not use session-based auth

### Authorization
- Voters: `ACCESS_GRANTED` overrides others only with unanimous strategy
- Security attributes on controllers: verify they match route config
- `IsGranted` attribute (PHP 8): compile-time check
- Role hierarchy: verify inheritance chain
- `denyAccessUnlessGranted()` throws `AccessDeniedException`

### CSRF Protection
- Forms must include `{{ csrf_token() }}` or use Form component
- API endpoints may legitimately skip CSRF (use token auth)
- `CsrfTokenManager`: verify token is validated, not just generated

## Forms & Validation

### Form Types
- Data transformer errors: verify expected input/output types
- `mapped => false` fields excluded from entity - handle manually
- Compound forms: nested form validation must pass
- Custom constraints: verify `validate()` logic handles edge cases

### Validation
- Constraint groups: verify correct group applied at each stage
- Cascading validation (`@Valid`): verify nested object is validated
- Custom validators: must handle null input gracefully
- Validation at entity level vs form level: check both

## HTTP & Routing

### Request Handling
- `$request->get()` checks multiple bags - use specific: `query->get()`, `request->get()`
- `$request->getContent()` for raw body (JSON APIs)
- File uploads: `$request->files->get()` returns `UploadedFile|null`
- Parameter converters: verify 404 on missing entity

### Response
- `JsonResponse`: ensure proper content type and encoding
- Streamed responses: verify headers sent before body
- Cache headers: verify no sensitive data cached publicly
- CORS headers: verify allowed origins are restrictive

## Event System

### Event Subscribers
- `getSubscribedEvents()` defines priority - verify order
- Stopping propagation affects subsequent listeners
- Kernel events: `kernel.request`, `kernel.response`, `kernel.exception`
- Console events for CLI command lifecycle

## Twig Templates

### Security
- Auto-escaping enabled by default
- `|raw` filter disables escaping - audit every usage
- `{% autoescape false %}` disables for entire block
- `{{ include() }}` with user-controlled template name = template injection
- `is_safe` on custom filters/functions disables escaping

## Console Commands

### Command Patterns
- `InputInterface` arguments/options are string type by default
- Use `$input->getArgument()` / `$input->getOption()`
- Long-running commands: handle signals with `SignalableCommandInterface`
- Lock commands with `LockableTrait` to prevent concurrent execution

## Messenger (Queue)

### Message Handling
- Handlers may receive messages multiple times (retry)
- Ensure idempotent handlers
- `HandlerFailedException` wraps original exception
- Async transport: message serialization must preserve all needed data
- Stamps: verify middleware doesn't strip required stamps

## Common Symfony Regressions

1. **Service not found**: Missing autowiring hint or interface binding
2. **Firewall bypass**: Incorrect access_control order
3. **Doctrine flush scope**: Unexpected entities persisted
4. **EntityManager closed**: Unhandled exception during flush
5. **Form CSRF**: Missing token in AJAX requests
6. **Voter logic**: Wrong strategy or incorrect vote return
7. **Twig raw output**: Using `|raw` on user-supplied data
8. **Command lock**: Missing lock on concurrent-unsafe commands
