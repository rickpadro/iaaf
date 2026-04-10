# Reference: Security Baseline Implementation

> Required for ALL projects. This is not optional. Load when implementing auth or on the first step that touches the database.

---

## 1. Row Level Security (RLS)

Every database query that returns or modifies user data MUST filter by the authenticated user's ID. No exceptions.

### PHP Vanilla (mysqli)

```php
// WRONG — returns any project by ID, no ownership check
$stmt = $conn->prepare("SELECT * FROM projects WHERE id = ?");
$stmt->bind_param("i", $projectId);

// CORRECT — always include user_id
$stmt = $conn->prepare("SELECT * FROM projects WHERE id = ? AND user_id = ?");
$stmt->bind_param("ii", $projectId, $_SESSION['user_id']);
```

**Pattern: Create a helper function used by ALL queries:**

```php
// lib/QueryHelper.php
class QueryHelper {
    public static function ownedBy($conn, $table, $id) {
        $userId = $_SESSION['user_id'];
        $stmt = $conn->prepare("SELECT * FROM {$table} WHERE id = ? AND user_id = ?");
        $stmt->bind_param("ii", $id, $userId);
        $stmt->execute();
        $result = $stmt->get_result()->fetch_assoc();
        if (!$result) {
            http_response_code(404);
            echo json_encode(['success' => false, 'error' => 'Not found']);
            exit;
        }
        return $result;
    }
}
```

### Laravel (Eloquent)

```php
// Option A: Global Scope (recommended — applies to ALL queries automatically)
// app/Models/Concerns/OwnedByUser.php
trait OwnedByUser {
    protected static function bootOwnedByUser() {
        static::addGlobalScope('owned', function ($query) {
            $query->where('user_id', auth()->id());
        });
        static::creating(function ($model) {
            $model->user_id = auth()->id();
        });
    }
}

// Usage in any model:
class Project extends Model {
    use OwnedByUser;
}
// Now Project::find(5) automatically adds WHERE user_id = auth()->id()
// Project::all() only returns the user's own projects

// Option B: Policy (for complex permissions)
// app/Policies/ProjectPolicy.php
public function view(User $user, Project $project) {
    return $user->id === $project->user_id;
}
```

### Next.js + Prisma/Drizzle

```typescript
// Middleware that injects userId into every query
// lib/db.ts
export function ownedQuery(userId: string) {
  return {
    projects: {
      findMany: (args = {}) => db.query.projects.findMany({
        ...args,
        where: { ...args.where, userId },
      }),
      findFirst: (args) => db.query.projects.findFirst({
        ...args,
        where: { ...args.where, userId },
      }),
    }
  }
}

// In route handler
const userDb = ownedQuery(session.user.id)
const projects = await userDb.projects.findMany()
```

### Supabase (PostgreSQL RLS)

```sql
-- Enable RLS on the table
ALTER TABLE projects ENABLE ROW LEVEL SECURITY;

-- Users can only see their own rows
CREATE POLICY "Users see own projects"
  ON projects FOR SELECT
  USING (user_id = auth.uid());

-- Users can only insert their own rows
CREATE POLICY "Users insert own projects"
  ON projects FOR INSERT
  WITH CHECK (user_id = auth.uid());

-- Users can only update their own rows
CREATE POLICY "Users update own projects"
  ON projects FOR UPDATE
  USING (user_id = auth.uid());

-- Users can only delete their own rows
CREATE POLICY "Users delete own projects"
  ON projects FOR DELETE
  USING (user_id = auth.uid());
```

### Tables that are EXEMPT from RLS

Some tables are intentionally shared (not user-specific):
- Configuration tables (platform_profiles, settings)
- Lookup tables (countries, categories)
- Admin-only tables

Document these exemptions explicitly. Every other table MUST have RLS.

---

## 2. CORS Configuration

### PHP Vanilla

```php
// lib/cors.php — include at the top of every API file
$allowed_origins = [
    'https://yourdomain.com',
    'https://app.yourdomain.com',
];

// In development, add localhost
if (getenv('APP_ENV') === 'development') {
    $allowed_origins[] = 'http://localhost';
    $allowed_origins[] = 'http://localhost:3000';
    $allowed_origins[] = 'http://localhost:8080';
}

$origin = $_SERVER['HTTP_ORIGIN'] ?? '';

if (in_array($origin, $allowed_origins)) {
    header("Access-Control-Allow-Origin: $origin");
    header("Access-Control-Allow-Credentials: true");
    header("Access-Control-Allow-Methods: GET, POST, PUT, PATCH, DELETE, OPTIONS");
    header("Access-Control-Allow-Headers: Content-Type, Authorization, X-Requested-With");
    header("Access-Control-Max-Age: 86400");
}

// Handle preflight
if ($_SERVER['REQUEST_METHOD'] === 'OPTIONS') {
    http_response_code(204);
    exit;
}
```

### Laravel

```php
// config/cors.php (already included in Laravel 11)
return [
    'paths' => ['api/*'],
    'allowed_origins' => [env('APP_URL')],
    'allowed_methods' => ['GET', 'POST', 'PUT', 'PATCH', 'DELETE'],
    'allowed_headers' => ['Content-Type', 'Authorization', 'X-Requested-With'],
    'supports_credentials' => true,
    'max_age' => 86400,
];
```

### Next.js

```typescript
// middleware.ts or next.config.js
// Same-origin by default (API routes + frontend on same domain)
// Only configure CORS if API is consumed by external frontends
```

### Apache (.htaccess)

```apache
<IfModule mod_headers.c>
    SetEnvIf Origin "^https://(yourdomain\.com|app\.yourdomain\.com)$" ORIGIN=$0
    Header set Access-Control-Allow-Origin "%{ORIGIN}e" env=ORIGIN
    Header set Access-Control-Allow-Credentials "true" env=ORIGIN
    Header set Access-Control-Allow-Methods "GET, POST, PUT, PATCH, DELETE, OPTIONS"
    Header set Access-Control-Allow-Headers "Content-Type, Authorization, X-Requested-With"
</IfModule>
```

### Rules

- **NEVER** use `Access-Control-Allow-Origin: *` with credentials
- Whitelist specific domains, not patterns
- Include `localhost` variants only in development
- Set `Max-Age` to cache preflight (86400 = 24 hours)

---

## 3. Security Headers

### PHP Vanilla

```php
// lib/security_headers.php — include in every page and API
header("Content-Security-Policy: default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; font-src 'self'; frame-ancestors 'none'");
header("X-Content-Type-Options: nosniff");
header("X-Frame-Options: DENY");
header("X-XSS-Protection: 1; mode=block");
header("Referrer-Policy: strict-origin-when-cross-origin");
header("Permissions-Policy: camera=(), microphone=(), geolocation=()");

// HSTS — only in production with HTTPS
if (isset($_SERVER['HTTPS']) && $_SERVER['HTTPS'] === 'on') {
    header("Strict-Transport-Security: max-age=63072000; includeSubDomains");
}
```

### Laravel

```php
// app/Http/Middleware/SecurityHeaders.php
public function handle($request, $next) {
    $response = $next($request);
    $response->headers->set('X-Content-Type-Options', 'nosniff');
    $response->headers->set('X-Frame-Options', 'DENY');
    $response->headers->set('X-XSS-Protection', '1; mode=block');
    $response->headers->set('Referrer-Policy', 'strict-origin-when-cross-origin');
    $response->headers->set('Permissions-Policy', 'camera=(), microphone=(), geolocation=()');
    $response->headers->set('Strict-Transport-Security', 'max-age=63072000; includeSubDomains');
    return $response;
}
// Register in bootstrap/app.php as global middleware
```

### Next.js

```javascript
// next.config.js
async headers() {
  return [{
    source: '/(.*)',
    headers: [
      { key: 'X-Content-Type-Options', value: 'nosniff' },
      { key: 'X-Frame-Options', value: 'DENY' },
      { key: 'Referrer-Policy', value: 'strict-origin-when-cross-origin' },
      { key: 'Permissions-Policy', value: 'camera=(), microphone=(), geolocation=()' },
      { key: 'Strict-Transport-Security', value: 'max-age=63072000; includeSubDomains' },
    ],
  }]
}
```

### Apache (.htaccess)

```apache
<IfModule mod_headers.c>
    Header set X-Content-Type-Options "nosniff"
    Header set X-Frame-Options "DENY"
    Header set X-XSS-Protection "1; mode=block"
    Header set Referrer-Policy "strict-origin-when-cross-origin"
    Header set Permissions-Policy "camera=(), microphone=(), geolocation=()"
    Header set Strict-Transport-Security "max-age=63072000; includeSubDomains"
    Header set Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; font-src 'self'; frame-ancestors 'none'"
</IfModule>
```

### CSP Adjustments by Project Type

| Project | CSP additions needed |
|---------|---------------------|
| Uses Google Fonts | `font-src 'self' fonts.googleapis.com fonts.gstatic.com` |
| Uses CDN for JS/CSS | `script-src 'self' cdn.jsdelivr.net; style-src 'self' cdn.jsdelivr.net` |
| Uses inline scripts (legacy) | `script-src 'self' 'unsafe-inline'` (try to eliminate) |
| Uses images from external sources | `img-src 'self' data: https:` |
| Embeds YouTube/Vimeo | `frame-src youtube.com vimeo.com` |
| Uses Stripe/PayPal | `script-src 'self' js.stripe.com; frame-src js.stripe.com` |

---

## 4. Rate Limiting

Limit how many requests a user or IP can make in a time window. Without this, attackers can brute-force login, scrape data, or overwhelm your server.

### Limits to Apply

| Endpoint | Limit | Why |
|----------|-------|-----|
| Login / Register / Password Reset | 5 req/min per IP | Prevent brute force |
| Authenticated API routes | 100 req/min per user | Prevent abuse |
| Public API routes | 20 req/min per IP | Prevent scraping |
| File uploads | 10 req/min per user | Prevent storage abuse |
| AI/LLM endpoints | 5 req/min per user | Prevent cost abuse |

### PHP Vanilla

```php
// lib/RateLimiter.php
class RateLimiter {
    private $conn;

    public function __construct($conn) {
        $this->conn = $conn;
    }

    /**
     * Check if request should be rate limited.
     * @param string $key  Identifier (IP for public, user_id for authenticated)
     * @param int $maxAttempts  Max requests allowed
     * @param int $windowSeconds  Time window in seconds (60 = per minute)
     * @return bool  true if allowed, false if rate limited
     */
    public function attempt($key, $maxAttempts = 5, $windowSeconds = 60) {
        $window = date('Y-m-d H:i:s', time() - $windowSeconds);

        // Count recent attempts
        $stmt = $this->conn->prepare(
            "SELECT COUNT(*) as cnt FROM rate_limits WHERE limiter_key = ? AND created_at > ?"
        );
        $stmt->bind_param("ss", $key, $window);
        $stmt->execute();
        $count = $stmt->get_result()->fetch_assoc()['cnt'];

        if ($count >= $maxAttempts) {
            return false; // Rate limited
        }

        // Record this attempt
        $stmt = $this->conn->prepare(
            "INSERT INTO rate_limits (limiter_key, created_at) VALUES (?, NOW())"
        );
        $stmt->bind_param("s", $key);
        $stmt->execute();

        return true; // Allowed
    }
}

// Table needed:
// CREATE TABLE rate_limits (
//     id INT AUTO_INCREMENT PRIMARY KEY,
//     limiter_key VARCHAR(255) NOT NULL,
//     created_at DATETIME NOT NULL,
//     INDEX idx_key_time (limiter_key, created_at)
// );
// Add cron to clean old records: DELETE FROM rate_limits WHERE created_at < DATE_SUB(NOW(), INTERVAL 1 HOUR);
```

**Usage in API:**

```php
// At the top of api/auth_login.php
require_once __DIR__ . '/../lib/RateLimiter.php';

$limiter = new RateLimiter($conn);
$ip = $_SERVER['REMOTE_ADDR'];

if (!$limiter->attempt("login:$ip", 5, 60)) {
    http_response_code(429);
    echo json_encode([
        'success' => false,
        'error' => 'Too many attempts. Try again in 1 minute.'
    ]);
    exit;
}
```

### Laravel

```php
// Already built-in. Just apply to routes:
// routes/web.php or routes/api.php
Route::post('/login', [LoginController::class, 'store'])
    ->middleware('throttle:5,1'); // 5 attempts per minute

Route::middleware(['auth', 'throttle:100,1'])->group(function () {
    // All authenticated API routes: 100 req/min
});

// Custom response for 429:
// app/Exceptions/Handler.php
protected function throttleException($request, $exception) {
    return response()->json([
        'error' => 'Too many requests. Try again later.',
        'retry_after' => $exception->getHeaders()['Retry-After'],
    ], 429);
}
```

### Next.js (Upstash Ratelimit)

```typescript
import { Ratelimit } from '@upstash/ratelimit'
import { Redis } from '@upstash/redis'

const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(5, '1 m'),
})

// In API route or middleware
const ip = request.headers.get('x-forwarded-for') ?? '127.0.0.1'
const { success, remaining } = await ratelimit.limit(ip)

if (!success) {
  return Response.json({ error: 'Too many requests' }, {
    status: 429,
    headers: { 'Retry-After': '60', 'X-RateLimit-Remaining': '0' }
  })
}
```

### Hono / Express

```typescript
// Hono
import { rateLimiter } from 'hono-rate-limiter'

app.use('/api/auth/*', rateLimiter({
  windowMs: 60 * 1000,
  limit: 5,
  keyGenerator: (c) => c.req.header('x-forwarded-for') || 'unknown',
}))

// Express
import rateLimit from 'express-rate-limit'

app.use('/api/auth', rateLimit({
  windowMs: 60 * 1000,
  max: 5,
  message: { error: 'Too many attempts. Try again in 1 minute.' }
}))
```

### Response Headers for Rate Limiting

Always include these headers so clients know their status:

```
X-RateLimit-Limit: 5
X-RateLimit-Remaining: 2
Retry-After: 45
```

---

## 5. API Keys in Environment Variables

API keys, database credentials, OAuth secrets, and encryption keys MUST live in environment variables. Never in code, never in database, never in logs.

### PHP Vanilla

```php
// config/app_config.php — load .env file
function loadEnv($path = __DIR__ . '/../.env') {
    if (!file_exists($path)) return;
    $lines = file($path, FILE_IGNORE_NEW_LINES | FILE_SKIP_EMPTY_LINES);
    foreach ($lines as $line) {
        if (strpos(trim($line), '#') === 0) continue;
        list($key, $value) = explode('=', $line, 2);
        $key = trim($key);
        $value = trim($value);
        // Remove quotes if present
        $value = trim($value, '"\'');
        putenv("$key=$value");
        $_ENV[$key] = $value;
    }
}

function env($key, $default = null) {
    return $_ENV[$key] ?? getenv($key) ?: $default;
}

// Load at app startup
loadEnv();

// Validate critical keys exist — fail fast
$required = ['DB_HOST', 'DB_DATABASE', 'DB_USERNAME'];
foreach ($required as $key) {
    if (!env($key)) {
        die("Missing required environment variable: $key");
    }
}
```

**Usage:**

```php
// CORRECT
$apiKey = env('ANTHROPIC_API_KEY');
$dbHost = env('DB_HOST');

// WRONG — never do this
$apiKey = 'sk-ant-abc123...';  // hardcoded in source
$dbHost = 'localhost';          // hardcoded in source
```

**.env file:**

```env
# Database
DB_HOST=127.0.0.1
DB_DATABASE=crm_app
DB_USERNAME=root
DB_PASSWORD=

# External APIs
ANTHROPIC_API_KEY=sk-ant-...
PERPLEXITY_API_KEY=pplx-...
STRIPE_SECRET_KEY=sk_test_...
RESEND_API_KEY=re_...

# OAuth
META_APP_ID=123456
META_APP_SECRET=abc123...
ENCRYPTION_KEY=random-32-char-string
```

**.env.example file** (this IS committed to git):

```env
# Database
DB_HOST=127.0.0.1
DB_DATABASE=your_database
DB_USERNAME=root
DB_PASSWORD=

# External APIs
ANTHROPIC_API_KEY=sk-ant-your-key
PERPLEXITY_API_KEY=pplx-your-key
STRIPE_SECRET_KEY=sk_test_your-key

# OAuth
META_APP_ID=your-app-id
META_APP_SECRET=your-app-secret
ENCRYPTION_KEY=generate-random-32-chars
```

**.gitignore:**

```
.env
```

### Laravel

Already built-in. `.env` is loaded automatically. Use `env('KEY')` or `config('services.stripe.secret')`.

### Next.js

Already built-in. `.env.local` for secrets. `NEXT_PUBLIC_` prefix for client-exposed vars. Validate with Zod at startup:

```typescript
const envSchema = z.object({
  DATABASE_URL: z.string().url(),
  STRIPE_SECRET_KEY: z.string().startsWith('sk_'),
})
export const env = envSchema.parse(process.env)
```

### Python

```python
# pip install python-dotenv pydantic-settings
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    database_url: str
    anthropic_api_key: str
    stripe_secret_key: str

    class Config:
        env_file = ".env"

settings = Settings()  # Crashes at startup if any field is missing
```

### Rules

1. **`.env` is in `.gitignore`.** Always. Non-negotiable.
2. **`.env.example` is committed** with placeholder values so new developers know what to set.
3. **Validate at startup.** If a critical key is missing, the app refuses to start.
4. **Never log API keys.** Even partially. `sk-ant-a...` in a log is still a risk.
5. **Different keys per environment.** Dev, staging, production each have their own.
6. **Rotate after team changes.** When someone leaves, rotate all keys they had access to.

---

## 6. SQL Injection Prevention

Every database query that includes user input MUST use parameterized queries. Never concatenate user input into SQL strings.

### The Attack

```
User input: ' OR 1=1 --

Vulnerable query:
  SELECT * FROM users WHERE email = '' OR 1=1 --'
  → Returns ALL users

User input: '; DROP TABLE users; --

Vulnerable query:
  SELECT * FROM users WHERE email = ''; DROP TABLE users; --'
  → Deletes the entire users table
```

### PHP Vanilla (mysqli) — CRITICAL

PHP vanilla is the highest risk because developers write SQL manually.

```php
// ████ DANGEROUS — SQL INJECTION ████
$email = $_POST['email'];
$sql = "SELECT * FROM users WHERE email = '$email'";
$result = $conn->query($sql);

// ████ DANGEROUS — even with escaping, this is fragile ████
$email = $conn->real_escape_string($_POST['email']);
$sql = "SELECT * FROM users WHERE email = '$email'";

// ✅ SAFE — parameterized query with prepared statement
$stmt = $conn->prepare("SELECT * FROM users WHERE email = ?");
$stmt->bind_param("s", $_POST['email']);
$stmt->execute();
$result = $stmt->get_result();
```

**Rules for PHP vanilla:**

```php
// Rule 1: NEVER use $conn->query() with user input
// Rule 2: ALWAYS use $conn->prepare() + bind_param()
// Rule 3: Type hints matter: "s" = string, "i" = integer, "d" = double

// Multiple parameters:
$stmt = $conn->prepare("SELECT * FROM tasks WHERE user_id = ? AND status = ? AND priority <= ?");
$stmt->bind_param("isi", $userId, $status, $maxPriority);

// INSERT:
$stmt = $conn->prepare("INSERT INTO clients (name, email, user_id) VALUES (?, ?, ?)");
$stmt->bind_param("ssi", $name, $email, $userId);
$stmt->execute();
$newId = $stmt->insert_id;

// UPDATE:
$stmt = $conn->prepare("UPDATE clients SET name = ?, email = ? WHERE id = ? AND user_id = ?");
$stmt->bind_param("ssii", $name, $email, $clientId, $userId);

// DELETE:
$stmt = $conn->prepare("DELETE FROM tasks WHERE id = ? AND user_id = ?");
$stmt->bind_param("ii", $taskId, $userId);

// LIKE search (the % goes in the variable, not in the SQL):
$search = "%{$_GET['q']}%";
$stmt = $conn->prepare("SELECT * FROM clients WHERE name LIKE ? AND user_id = ?");
$stmt->bind_param("si", $search, $userId);

// IN clause (dynamic number of params):
$ids = [1, 2, 3, 4];
$placeholders = implode(',', array_fill(0, count($ids), '?'));
$types = str_repeat('i', count($ids));
$stmt = $conn->prepare("SELECT * FROM tasks WHERE id IN ($placeholders) AND user_id = ?");
$stmt->bind_param($types . 'i', ...[...$ids, $userId]);
```

### Laravel (Eloquent) — Low Risk

Eloquent parameterizes automatically. Only risk is `DB::raw()`:

```php
// ✅ SAFE — Eloquent handles parameterization
User::where('email', $request->email)->first();
Task::where('user_id', auth()->id())->where('status', 'active')->get();

// ✅ SAFE — Query builder with bindings
DB::table('users')->where('email', $request->email)->first();

// ████ DANGEROUS — raw SQL with user input
DB::select("SELECT * FROM users WHERE email = '{$request->email}'");

// ✅ SAFE — raw SQL with bindings
DB::select("SELECT * FROM users WHERE email = ?", [$request->email]);
```

### Next.js + Prisma/Drizzle — Low Risk

ORMs parameterize automatically. Only risk is raw SQL:

```typescript
// ✅ SAFE — Prisma
await prisma.user.findFirst({ where: { email: input.email } })

// ✅ SAFE — Drizzle
await db.select().from(users).where(eq(users.email, input.email))

// ████ DANGEROUS — raw SQL
await prisma.$queryRawUnsafe(`SELECT * FROM users WHERE email = '${input}'`)

// ✅ SAFE — raw SQL with parameters
await prisma.$queryRaw`SELECT * FROM users WHERE email = ${input}`
```

### Input Validation BEFORE the Query

Defense in depth: validate the input type before it even reaches the query.

```php
// PHP: cast to expected type
$id = (int) $_GET['id'];          // Forces integer, kills injection
$email = filter_var($_POST['email'], FILTER_VALIDATE_EMAIL); // Validates format
if (!$email) { http_response_code(400); exit; }

// Also validate with regex for specific patterns:
$slug = preg_match('/^[a-z0-9\-]+$/', $_GET['slug']) ? $_GET['slug'] : null;
```

```typescript
// TypeScript: Zod validation at API boundary
const schema = z.object({
  id: z.coerce.number().int().positive(),
  email: z.string().email(),
  slug: z.string().regex(/^[a-z0-9-]+$/),
})
const input = schema.parse(req.body) // Throws if invalid
```

### Test for SQL Injection

After implementing, test these inputs in every form and API parameter:

```
' OR 1=1 --
'; DROP TABLE users; --
' UNION SELECT * FROM users --
1; SELECT * FROM information_schema.tables --
```

If any of these returns data or causes an error with SQL details, you have a vulnerability.
