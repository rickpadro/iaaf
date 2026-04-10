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
