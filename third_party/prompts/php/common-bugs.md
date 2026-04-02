# Common Bug Patterns in PHP

## SQL Injection

### Raw Query Concatenation

**Bug**: Concatenating user input into SQL strings.

**Example (BAD)**:
```php
$results = DB::select("SELECT * FROM users WHERE email = '" . $email . "'");
$stmt = $pdo->query("SELECT * FROM orders WHERE id = " . $_GET['id']);
```

**Example (GOOD)**:
```php
$results = DB::select("SELECT * FROM users WHERE email = ?", [$email]);
$stmt = $pdo->prepare("SELECT * FROM orders WHERE id = ?");
$stmt->execute([$_GET['id']]);
```

### ORM Raw Methods

**Bug**: Using raw query methods with unparameterized input.

**Example (BAD)**:
```php
User::whereRaw("email = '$email'")->first();
DB::raw("COUNT(*) as count WHERE status = '$status'");
Order::selectRaw("*, $column as alias")->get();
```

**Example (GOOD)**:
```php
User::whereRaw("email = ?", [$email])->first();
DB::raw("COUNT(*) as count WHERE status = ?", [$status]);
// For column names, validate against whitelist
$allowed = ['total', 'quantity', 'price'];
if (!in_array($column, $allowed)) throw new \InvalidArgumentException();
```

## XSS (Cross-Site Scripting)

### Unescaped Output

**Bug**: Outputting user data without escaping.

**Example (BAD)**:
```php
echo "Welcome, " . $_GET['name'];
echo "<a href='" . $user->website . "'>";
{!! $user->bio !!}  // Blade raw output
```

**Example (GOOD)**:
```php
echo "Welcome, " . htmlspecialchars($_GET['name'], ENT_QUOTES, 'UTF-8');
echo "<a href='" . htmlspecialchars($user->website, ENT_QUOTES, 'UTF-8') . "'>";
{{ $user->bio }}  // Blade auto-escapes
```

## Type Juggling Bugs

### Loose Comparison

**Bug**: Using `==` instead of `===` for security-sensitive comparisons.

**Example (BAD)**:
```php
if ($token == $expectedToken) { /* authenticate */ }
// "0e123" == "0e456" is TRUE (scientific notation comparison)

if (in_array($role, $allowedRoles)) { /* authorize */ }
// in_array(0, ["admin"]) is TRUE with loose comparison
```

**Example (GOOD)**:
```php
if (hash_equals($expectedToken, $token)) { /* authenticate */ }
// hash_equals is timing-safe and strict

if (in_array($role, $allowedRoles, true)) { /* authorize */ }
// Third parameter true enables strict comparison
```

### strpos() Return Value

**Bug**: Using falsy check on `strpos()` return.

**Example (BAD)**:
```php
if (!strpos($haystack, $needle)) {
    // BUG: This is true when $needle is at position 0!
}
```

**Example (GOOD)**:
```php
if (strpos($haystack, $needle) === false) {
    // Correct: explicitly check for false
}
// Or better (PHP 8.0+):
if (!str_contains($haystack, $needle)) {
    // Modern alternative
}
```

## Null Reference Errors

### Accessing Nullable Relations

**Bug**: Accessing properties on nullable ORM relations.

**Example (BAD)**:
```php
$user->profile->avatar;          // profile can be null (hasOne)
$order->latestShipment->status;  // latestShipment can be null
```

**Example (GOOD)**:
```php
$user->profile?->avatar;                    // Nullsafe operator
$order->latestShipment?->status ?? 'none';  // With default
```

### find() vs findOrFail()

**Bug**: Treating `find()` result as non-nullable.

**Example (BAD)**:
```php
$user = User::find($id);
$user->update(['name' => $name]);  // $user might be null!
```

**Example (GOOD)**:
```php
$user = User::findOrFail($id);    // Throws ModelNotFoundException
$user->update(['name' => $name]);
```

## Resource Leaks

### Uncommitted Transactions

**Bug**: Missing rollback on exception in transaction.

**Example (BAD)**:
```php
DB::beginTransaction();
$order = Order::create($data);
$payment = Payment::process($order);  // May throw!
DB::commit();
// If Payment::process() throws, transaction is left open
```

**Example (GOOD)**:
```php
DB::beginTransaction();
try {
    $order = Order::create($data);
    $payment = Payment::process($order);
    DB::commit();
} catch (\Throwable $e) {
    DB::rollBack();
    throw $e;
}
// Or use the framework helper:
DB::transaction(function () use ($data) {
    $order = Order::create($data);
    Payment::process($order);
});
```

### Unclosed File Handles

**Bug**: Not closing file handles on error paths.

**Example (BAD)**:
```php
$fh = fopen($path, 'r');
$data = processFile($fh);  // May throw!
fclose($fh);               // Never reached on exception
```

**Example (GOOD)**:
```php
$fh = fopen($path, 'r');
try {
    $data = processFile($fh);
} finally {
    fclose($fh);
}
```

## Command Injection

### Unescaped Shell Arguments

**Bug**: Passing user input to shell functions without escaping.

**Example (BAD)**:
```php
exec("convert " . $uploadedFile . " output.png");
system("grep " . $searchTerm . " /var/log/app.log");
$output = `ls $directory`;
```

**Example (GOOD)**:
```php
exec("convert " . escapeshellarg($uploadedFile) . " output.png");
// Or avoid shell entirely:
$lines = file('/var/log/app.log');
$matches = preg_grep('/' . preg_quote($searchTerm, '/') . '/', $lines);
```

## Mass Assignment

### Unguarded Mass Assignment

**Bug**: Allowing mass assignment of sensitive fields.

**Example (BAD)**:
```php
// Model has no $fillable or $guarded
User::create($request->all());
// Attacker can set: is_admin=1, role=admin, balance=999999

$user->update($request->all());
```

**Example (GOOD)**:
```php
User::create($request->only(['name', 'email', 'password']));
// Or define $fillable on the model:
// protected $fillable = ['name', 'email', 'password'];
```

## Deserialization

### Unsafe unserialize()

**Bug**: Using `unserialize()` with untrusted data.

**Example (BAD)**:
```php
$data = unserialize($_COOKIE['preferences']);
$obj = unserialize(file_get_contents($cacheFile));
```

**Example (GOOD)**:
```php
$data = json_decode($_COOKIE['preferences'], true);
// If unserialize is required:
$obj = unserialize($data, ['allowed_classes' => [UserPrefs::class]]);
```

## Path Traversal

### Unvalidated File Paths

**Bug**: Using user input in file paths without validation.

**Example (BAD)**:
```php
$file = file_get_contents('/uploads/' . $_GET['filename']);
include('/templates/' . $request->input('template') . '.php');
```

**Example (GOOD)**:
```php
$filename = basename($_GET['filename']);  // Strip directory components
$file = file_get_contents('/uploads/' . $filename);

// For template inclusion, use whitelist
$allowed = ['home', 'about', 'contact'];
$template = $request->input('template');
if (!in_array($template, $allowed, true)) {
    abort(404);
}
include('/templates/' . $template . '.php');
```

## Session Bugs

### Missing Session Regeneration

**Bug**: Not regenerating session ID after authentication state change.

**Example (BAD)**:
```php
if ($this->attemptLogin($credentials)) {
    // Session ID not regenerated - session fixation vulnerability!
    return redirect('/dashboard');
}
```

**Example (GOOD)**:
```php
if ($this->attemptLogin($credentials)) {
    session_regenerate_id(true);  // Or framework equivalent
    // Laravel: $request->session()->regenerate();
    return redirect('/dashboard');
}
```

## CSRF Bugs

### Missing CSRF Token Verification

**Bug**: State-changing endpoints without CSRF protection.

**Example (BAD)**:
```php
// Route without CSRF middleware
Route::post('/transfer', [BankController::class, 'transfer'])
    ->withoutMiddleware(['csrf']);

// Form without CSRF token
<form method="POST" action="/settings">
    <input name="email" value="...">
    <button type="submit">Save</button>
</form>
```

**Example (GOOD)**:
```php
<form method="POST" action="/settings">
    @csrf
    <input name="email" value="...">
    <button type="submit">Save</button>
</form>
```

## Encoding Bugs

### Byte vs Character Operations

**Bug**: Using byte-oriented string functions on multibyte strings.

**Example (BAD)**:
```php
$len = strlen($name);        // Counts bytes, not characters
$sub = substr($name, 0, 10); // May cut multibyte character
$low = strtolower($name);    // Fails for non-ASCII
```

**Example (GOOD)**:
```php
$len = mb_strlen($name);           // Counts characters
$sub = mb_substr($name, 0, 10);    // Respects character boundaries
$low = mb_strtolower($name);       // Unicode-aware
```
