# Building Block: Security & Observability Patterns

Covers: OWASP basics, CORS, CSP, rate limiting, input validation, secrets management, error tracking, logging, monitoring, and health checks.

---

## SECURITY

### OWASP Top Risks by Project Type

| Risk | SaaS | API | Marketing | Mobile | Internal |
|------|------|-----|-----------|--------|----------|
| Injection (SQL, command) | Critical | Critical | Low | Medium | Critical |
| Broken Auth | Critical | Critical | Low | Critical | High |
| Broken Access Control | Critical | Critical | Low | Critical | Critical |
| XSS | Critical | Low | Medium | Low | High |
| Security Misconfiguration | High | High | Medium | Medium | High |

### CORS

```typescript
// API: only allow your own frontend. NEVER use '*' with credentials.
cors({
  origin: ['https://app.yourdomain.com'],
  allowMethods: ['GET', 'POST', 'PATCH', 'DELETE'],
  credentials: true,
})
```

Not needed if frontend and API share the same domain (Next.js API routes).

### Security Headers (Production)

```typescript
'Content-Security-Policy': "default-src 'self'; frame-ancestors 'none'"
'X-Content-Type-Options': 'nosniff'
'X-Frame-Options': 'DENY'
'Referrer-Policy': 'strict-origin-when-cross-origin'
'Strict-Transport-Security': 'max-age=63072000; includeSubDomains'
```

### Rate Limiting

| Endpoint | Limit | Why |
|----------|-------|-----|
| Auth (login, register) | 5-10 req/min per IP | Prevent brute force |
| API (authenticated) | 100-1000 req/min per user | Prevent abuse |
| API (public) | 20-60 req/min per IP | Prevent scraping |
| File uploads | 10 req/min per user | Prevent storage abuse |

Use **Upstash Ratelimit** (serverless) or **hono-rate-limiter** (traditional).

### Input Validation

> Validate at the boundary. Sanitize on output. Never trust user input.

- **Zod** for all API inputs at the route handler level
- React auto-escapes JSX (XSS safe by default)
- Never interpolate user input into SQL — use parameterized queries (Prisma/Drizzle do this automatically)
- Validate file types on both client AND server

### Secrets Management

1. **Never commit secrets to git.** Not even in "private" repos.
2. **Validate env vars at startup with Zod** — fail fast if missing.
3. **Use `.env.example`** with placeholder values, `.env` in `.gitignore`.
4. **Rotate secrets** after team member departures.
5. **Different secrets per environment** (dev, staging, production).

```typescript
// lib/env.ts — crashes at startup if any secret is missing
const envSchema = z.object({
  DATABASE_URL: z.string().url(),
  CLERK_SECRET_KEY: z.string().startsWith('sk_'),
  STRIPE_SECRET_KEY: z.string().startsWith('sk_'),
})
export const env = envSchema.parse(process.env)
```

### Audit Logging (SaaS and Internal Tools)

Log: user logins, permission changes, data modifications, data deletions, admin actions. Store in `audit_logs` table with user_id, action, resource, details (JSONB), IP, timestamp.

---

## OBSERVABILITY

### Error Tracking

| Solution | Best For | Cost |
|----------|----------|------|
| **Sentry** | Most projects | Free 5k events/mo, $26/mo for 50k |
| **LogRocket** | Frontend debugging (session replay) | Free 1k sessions/mo, $99/mo |
| **Highlight.io** | Open-source alternative | Free 500 sessions/mo |

**Recommendation:** Sentry for all production projects. Even MVPs — the free tier is generous enough.

### Sentry Setup (Next.js)
```bash
npx @sentry/wizard@latest -i nextjs
```
Configure: `tracesSampleRate: 0.1` (10%), `replaysOnErrorSampleRate: 1.0` (100% of errors).

### Logging

| Solution | When |
|----------|------|
| **Pino** | Production Node.js (fastest JSON logger) |
| **Console.log** | MVPs and prototypes (seriously, it's fine) |
| **Axiom** | Log aggregation (generous free tier, Vercel integration) |

```typescript
import pino from 'pino'
export const logger = pino({ level: process.env.LOG_LEVEL || 'info' })

logger.info({ userId, action: 'login' }, 'User logged in')
logger.error({ err, requestId }, 'Payment failed')
```

**Log levels:** error (broken), warn (concerning), info (business events), debug (troubleshooting).

**Never log:** passwords, tokens, API keys, credit card numbers, full request bodies.

### Health Check Endpoint

Every production service needs `/api/health`:

```typescript
export async function GET() {
  try {
    await db.execute(sql`SELECT 1`)
    return Response.json({ status: 'healthy', timestamp: new Date().toISOString() })
  } catch {
    return Response.json({ status: 'unhealthy' }, { status: 503 })
  }
}
```

### Monitoring

| Solution | When |
|----------|------|
| **Vercel Analytics** | Vercel-hosted, zero config |
| **Plausible** | Privacy-first web analytics |
| **BetterStack** | Uptime monitoring + status pages |

### Recommendation by Project Type

| Project | Error Tracking | Logging | Monitoring |
|---------|---------------|---------|------------|
| MVP | Console + Vercel logs | Console.log | Vercel Analytics |
| SaaS (production) | **Sentry** | **Pino** + Axiom | Vercel Analytics + BetterStack |
| API / Backend | **Sentry** | **Pino** + Axiom | BetterStack uptime |
| Marketing Site | Skip | Not needed | Plausible |

## Common Pitfalls

- **Don't roll your own auth.** Use Clerk, NextAuth, or Supabase Auth.
- **Don't expose stack traces in production.** Generic errors to client, full logs server-side.
- **Don't log everything.** Meaningful events at the right level. Noisy logs are worse than no logs.
- **Don't skip error tracking for MVPs.** Sentry free tier costs nothing.
- **Don't sample 100% of traces.** Start at 10%, increase if needed.
- **Don't forget alerts.** Monitoring without alerts is just data collection.
