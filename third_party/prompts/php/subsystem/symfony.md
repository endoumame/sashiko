# Symfony Subsystem Guide

## Doctrine ORM
- Entity lifecycle: `@PrePersist`, `@PostUpdate` callbacks
- Flush writes ALL pending changes at once
- EntityManager becomes closed after unhandled exception during flush
- `orphanRemoval=true` deletes unreferenced children
- Native queries need parameter binding

## Security
- Firewall access_control rules: first match wins - ORDER MATTERS
- Voters: ACCESS_GRANTED overrides depends on strategy
- `IsGranted` attribute for controller-level auth
- CSRF: forms need `{{ csrf_token() }}` or Form component

## Services
- Autowiring resolves type-hinted constructor params
- Singleton by default within request scope
- Don't resolve services in register() of compiler passes

## Twig
- Auto-escapes by default; `|raw` disables - audit every usage
- `{% autoescape false %}` disables for block
- User-controlled template names = template injection

## Common Regressions
1. Firewall bypass: wrong access_control order
2. Doctrine flush scope: unexpected entities persisted
3. EntityManager closed: unhandled exception during flush
4. Twig raw output: `|raw` on user data
