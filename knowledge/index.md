# Knowledge Index

Read this file FIRST in Phase 2 and Phase 4 to decide which files to load. Only read the files you need.

## Archetypes (read ONE based on classification)

| File | When to Read |
|------|-------------|
| `archetypes/saas-webapp.md` | Web app with user accounts, subscriptions, dashboards |
| `archetypes/marketing-site.md` | Landing pages, portfolios, static sites with forms |
| `archetypes/mobile-app.md` | iOS/Android apps with React Native/Expo |
| `archetypes/api-backend.md` | REST/GraphQL APIs, microservices, backend services (includes Python/Go stacks) |
| `archetypes/internal-tool.md` | Admin panels, dashboards, team-only tools |
| `archetypes/content-platform.md` | Blogs, docs, CMS, content-driven sites |
| `archetypes/e-commerce.md` | Online stores, product catalogs, checkout flows |
| `archetypes/ai-ml-app.md` | Chat AI, RAG, document Q&A, LLM-powered features |
| `archetypes/cli-tool.md` | Command-line tools, terminal utilities, scaffolders |
| `archetypes/realtime-collab.md` | Collaborative editing, live cursors, real-time sync |

## Building Blocks (read ONLY those relevant to the project)

| File | Contains | Read When |
|------|----------|-----------|
| `building-blocks/auth-patterns.md` | Clerk vs NextAuth vs Supabase Auth vs Firebase. Roles, sessions, social login. | Project has user accounts |
| `building-blocks/database-patterns.md` | Postgres vs MySQL vs MongoDB vs SQLite. Prisma vs Drizzle. Schema conventions, migrations. | Project has a database |
| `building-blocks/api-design-patterns.md` | REST vs tRPC vs GraphQL vs Server Actions. Response shapes, pagination, validation. | Project exposes or consumes an API |
| `building-blocks/frontend-patterns.md` | Next.js vs Astro vs Nuxt. Tailwind v4 setup, design tokens, shadcn/ui. Server vs Client Components. Zustand, TanStack Query. | Project has a frontend UI |
| `building-blocks/testing-patterns.md` | Vitest, Playwright, testing strategy by project type. What to test and what to skip. | Any production project |
| `building-blocks/payments-patterns.md` | Stripe vs Lemonsqueezy. Subscriptions, webhooks, billing portal, pricing page patterns. | Project has payments or billing |
| `building-blocks/integrations-patterns.md` | Email (Resend, SendGrid), push notifications, in-app notifications, file uploads (Uploadthing, S3, Cloudinary), CDN. | Project sends emails, notifications, or handles file uploads |
| `building-blocks/security-ops-patterns.md` | OWASP checklist, CORS, CSP, rate limiting, input validation, secrets. Error tracking (Sentry), logging (Pino), monitoring, health checks. | Any production project |
| `building-blocks/deployment-patterns.md` | Vercel vs Railway vs Fly.io vs AWS. CI/CD with GitHub Actions. Branch strategies, preview deploys, env management, releases. | Any project that deploys |
| `building-blocks/i18n-patterns.md` | next-intl, locale routing, translation files, RTL support, date/number formatting. | Project supports multiple languages |
| `building-blocks/performance-patterns.md` | Core Web Vitals, image optimization, bundle size, caching strategies, DB query optimization. | Performance-critical or public-facing project |
| `building-blocks/claude-behavior-patterns.md` | The Foreman rules: universal behavior rules + per-archetype additions. Build efficiency guidelines. | ALWAYS in Phase 4 (Generate) |

## Other Knowledge (read as needed)

| File | When to Read |
|------|-------------|
| `stack-compatibility.md` | Phase 3 — validating the proposed tech stack. Contains proven combos and anti-patterns. |
| `hybrid-patterns.md` | Phase 1 — when project spans multiple archetypes. How to merge stacks and build orders. |
| `blueprint-checklist.md` | Phase 4 — QA gate before saving the blueprint. |
| `skills-registry.md` | Phase 4 — mapping Claude Code skills to blueprint sections. |
