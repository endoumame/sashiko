# PHP Subsystem / Framework Detection

## Trigger Rules

Scan the diff for these patterns and load the corresponding subsystem files.

| Trigger Pattern | File to Load |
|----------------|--------------|
| `app/`, `Illuminate\`, Eloquent, Blade, Artisan, `routes/` | `laravel.md` |
| `src/`, `Symfony\Component\`, Doctrine, Twig, `config/services` | `symfony.md` |
| `wp_`, `add_action`, `add_filter`, `$wpdb`, `wp-content/` | `wordpress.md` |
| `composer.json`, `autoload`, PSR-4, PSR-12 | `composer.md` |
| PDO, MySQLi, query, migration, schema | `database.md` |
| `test/`, `tests/`, PHPUnit, Pest, `@test`, `@dataProvider` | `testing.md` |
| REST, API, JSON response, `JsonResponse`, endpoint | `api.md` |
| `queue`, `job`, `dispatch`, `ShouldQueue`, worker | `queue.md` |
| `auth`, `middleware`, `guard`, `policy`, `gate`, `voter` | `auth.md` |
| `cache`, `redis`, `memcached`, `Cache::`, session | `caching.md` |

## Multiple Matches

If the diff touches multiple subsystems, load ALL matching files.
Order of loading does not matter.

## No Match

If no subsystem triggers match, the code is generic PHP. The core
`technical-patterns.md` provides sufficient context for review.
