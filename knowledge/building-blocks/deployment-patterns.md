# Building Block: Deployment & CI/CD Patterns

Covers: hosting platforms, CI/CD pipelines, branch strategies, preview deploys, environment management, and releases.

## Hosting Decision Matrix

| Platform | Best For | Pros | Cons | Cost |
|----------|----------|------|------|------|
| **Vercel** | Next.js, Astro, frontend | Zero-config, preview deploys, edge | Expensive at scale | Free hobby, $20/mo Pro |
| **Railway** | Full-stack, backend, Docker | Easy Docker deploys, databases included | Less edge network | $5/mo + usage |
| **Fly.io** | Global edge, Docker | Multi-region, low latency | Steeper learning curve | Free tier, pay per use |
| **Cloudflare Pages** | Static, Workers | Fastest edge, free | Limited server-side | Generous free tier |
| **AWS (ECS/Lambda)** | Enterprise, complex infra | Full control | Complex, expensive knowledge | Pay per use |

### Recommendation by Project Type

| Project | Host | Why |
|---------|------|-----|
| Next.js / Astro (frontend) | **Vercel** | Built by same team, zero-config |
| API / Backend (Node, Python, Go) | **Railway** or **Fly.io** | Need persistent process |
| Static / Marketing | **Vercel** or **Cloudflare Pages** | CDN, fast, free |
| Mobile | **EAS Build** (Expo) | Integrated build + OTA |

## Branch Strategy

| Strategy | Best For |
|----------|----------|
| **GitHub Flow** (recommended) | Most teams. Feature branches from `main` → PR → review → merge → deploy. |
| **Trunk-Based** | Small teams, fast iteration. Everyone commits to `main`. |
| **Git Flow** | Versioned releases, enterprise. Complex but structured. |

## CI/CD Pipeline by Project Type

### SaaS / Web App
```
PR → lint + type-check + unit tests + Vercel preview deploy → review
Merge to main → tests again → Vercel production deploy (automatic)
```

### API / Backend
```
PR → lint + type-check + unit tests + integration tests (test DB) → review
Merge to main → all tests → run DB migrations → Railway/Fly deploy → health check
```

### Marketing Site
```
PR → lint + build + Lighthouse CI + Vercel preview → review
Merge to main → build + deploy (automatic)
```

### Mobile App
```
PR → lint + type-check + unit tests + EAS preview build → review
Merge to main → all tests → EAS production build → submit to stores (manual)
```

## GitHub Actions: Basic CI

```yaml
name: CI
on:
  push: { branches: [main] }
  pull_request: { branches: [main] }
jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20, cache: 'pnpm' }
      - run: pnpm install --frozen-lockfile
      - run: pnpm lint
      - run: pnpm type-check
      - run: pnpm test
```

Add Playwright E2E on PRs, DB services for integration tests, and Changesets for library releases as needed.

## Preview Deploys

| Platform | How |
|----------|-----|
| **Vercel** | Automatic on PR push, comments preview URL |
| **Netlify** | Automatic on PR push |
| **Railway** | Enable PR environments for full-stack preview |

Preview deploys = QA without local setup. Reviewers see the running app, not just code.

## Environment Management

### Minimum Variables (every project)
```env
DATABASE_URL=             # Database connection string
NEXT_PUBLIC_APP_URL=      # Public app URL
```

### Common Variables by Service
```env
# Auth (Clerk)
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=
CLERK_SECRET_KEY=

# Auth (Supabase)
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=

# Payments (Stripe)
STRIPE_SECRET_KEY=
STRIPE_WEBHOOK_SECRET=

# Email (Resend)
RESEND_API_KEY=
```

### Rules
- NEVER commit `.env` files. Use `.env.example` with placeholders.
- `NEXT_PUBLIC_` prefix for client-exposed vars in Next.js.
- Validate all env vars at startup with Zod.
- Different secrets per environment (dev, staging, production).
- Use platform secret management in production (Vercel env vars, Railway variables).

## Release Management

**Web apps:** Merge to `main` = deploy. No versioning needed. Rollback = revert commit.

**Libraries / CLIs:** Semantic versioning (MAJOR.MINOR.PATCH) with Changesets for automated releases.

## Common Pitfalls

- **Don't skip CI for small changes.** Small changes break builds.
- **Don't make CI take > 10 minutes.** Cache deps, parallelize tests.
- **Don't run E2E on every commit.** Run on PR + merge to main.
- **Don't test against production databases in CI.** Use test DB or containers.
- **Don't ignore flaky tests.** Fix or remove them.
