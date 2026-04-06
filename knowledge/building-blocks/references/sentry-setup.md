# Reference: Sentry Setup Patterns

> Supplementary detail for security-ops-patterns.md. Load only when implementing error tracking.

## Next.js Setup

```bash
npx @sentry/wizard@latest -i nextjs
```

Auto-configures: `sentry.client.config.ts`, `sentry.server.config.ts`, `sentry.edge.config.ts`, `next.config.js`.

```typescript
// sentry.client.config.ts
Sentry.init({
  dsn: process.env.NEXT_PUBLIC_SENTRY_DSN,
  tracesSampleRate: 0.1,         // 10% of transactions
  replaysSessionSampleRate: 0,    // Don't record all sessions
  replaysOnErrorSampleRate: 1.0,  // Record 100% of sessions with errors
  environment: process.env.NODE_ENV,
})
```

## Laravel Setup

```bash
composer require sentry/sentry-laravel
php artisan sentry:publish --dsn=YOUR_DSN
```

```php
// config/sentry.php
'dsn' => env('SENTRY_LARAVEL_DSN'),
'traces_sample_rate' => 0.1,
'profiles_sample_rate' => 0.1,
```

```php
// app/Exceptions/Handler.php
public function register()
{
    $this->reportable(function (Throwable $e) {
        if (app()->bound('sentry')) {
            app('sentry')->captureException($e);
        }
    });
}
```

## React Native / Expo Setup

```bash
npx expo install @sentry/react-native
```

```typescript
import * as Sentry from '@sentry/react-native'

Sentry.init({
  dsn: 'YOUR_DSN',
  tracesSampleRate: 0.2,
  enableAutoSessionTracking: true,
})
```

## Structured Logging with Pino

```typescript
import pino from 'pino'

export const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  transport: process.env.NODE_ENV === 'development'
    ? { target: 'pino-pretty' }
    : undefined,
})

// Usage
logger.info({ userId, action: 'login' }, 'User logged in')
logger.error({ err, requestId }, 'Payment processing failed')
logger.warn({ usage: 95 }, 'User approaching plan limit')
```

## Health Check Endpoint

```typescript
// /api/health/route.ts
export async function GET() {
  try {
    await db.execute(sql`SELECT 1`)
    return Response.json({
      status: 'healthy',
      timestamp: new Date().toISOString(),
      version: process.env.APP_VERSION || 'unknown',
    })
  } catch (error) {
    return Response.json({ status: 'unhealthy', error: error.message }, { status: 503 })
  }
}
```
