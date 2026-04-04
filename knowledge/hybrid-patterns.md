# Hybrid Project Patterns

Many projects span multiple archetypes. A SaaS app needs a marketing landing page. An API needs an admin dashboard. A content platform needs a payment system. This guide explains how to handle projects that don't fit neatly into one archetype.

---

## Common Hybrid Combinations

| Primary | Secondary | Example | Framework |
|---------|-----------|---------|-----------|
| SaaS Web App | Marketing Site | SaaS with public landing + pricing pages | Next.js (route groups: `(app)` + `(marketing)`) |
| API Backend | Internal Tool | API with admin dashboard | Hono (API) + Next.js (admin) or monorepo |
| Content Platform | Marketing Site | Blog with product landing page | Next.js or Astro (both handle content + marketing) |
| Mobile App | API Backend | Mobile app with custom backend | Expo (mobile) + Hono/FastAPI (backend) |
| E-Commerce | Content Platform | Online store with a blog | Next.js (route groups for shop + blog) |
| SaaS Web App | AI/ML App | SaaS with AI-powered features | Next.js + Vercel AI SDK |

---

## How to Identify Hybrids (Phase 1)

During discovery, watch for signals that the project spans archetypes:

- User describes both a **public-facing site AND an authenticated app** → SaaS + Marketing
- User needs **an API AND a way to manage it** → API + Internal Tool
- User wants **a mobile app AND needs the backend built too** → Mobile + API
- User mentions **AI features inside a larger product** → Primary archetype + AI/ML
- User describes **selling products AND publishing content** → E-Commerce + Content

**Classification rule:** Always pick the **primary** archetype (the core value of the product) and note the secondary. The primary determines the base stack and directory structure. The secondary adds features into that structure.

---

## How to Merge Architectures

### Directory Structure Merge Strategy

**Same Framework (most common — Next.js):**
Use route groups to separate concerns:

```
src/app/
  (marketing)/           # Public pages — from Marketing archetype
    page.tsx             # Landing page
    pricing/page.tsx
    blog/                # From Content archetype
    layout.tsx           # Marketing layout (navbar + footer)
  (app)/                 # Authenticated app — from SaaS/Internal archetype
    dashboard/page.tsx
    [feature]/page.tsx
    settings/page.tsx
    layout.tsx           # App layout (sidebar + topbar)
  (admin)/               # Admin panel — from Internal Tool archetype
    users/page.tsx
    analytics/page.tsx
    layout.tsx           # Admin layout
  api/
    webhooks/            # Webhooks from all archetypes
    [feature]/route.ts   # API routes
```

**Different Frameworks (e.g., Mobile + API):**
Use a monorepo:

```
apps/
  mobile/                # Expo app (from Mobile archetype)
  api/                   # Hono/FastAPI (from API archetype)
  admin/                 # Next.js (from Internal Tool archetype, if needed)
packages/
  shared/                # Shared types, validation schemas
  config/                # Shared configuration (ESLint, TypeScript)
```

### Stack Merge Rules

1. **One framework for web.** Don't use Next.js AND Astro in the same project. Pick the one that covers the primary archetype.
2. **One database.** Share the same database across all parts. Add tables/schemas as needed.
3. **One auth provider.** Auth is shared across all parts of the app.
4. **Separate concerns via route groups or packages**, not separate deployments (unless the secondary is a truly independent service like an API).

---

## Build Order Merge Strategy

The build order for a hybrid project interleaves steps from both archetypes. The pattern:

1. **Shared infrastructure first** (Steps 1-3): scaffolding, database, auth — these are shared
2. **Primary archetype features** (Steps 4-8): core feature of the primary archetype
3. **Secondary archetype features** (Steps 9-11): features from the secondary archetype
4. **Polish and deploy** (Steps 12-14): shared across all parts

### Example: SaaS + Marketing Site

```
Step 1:  Scaffolding (Next.js, Tailwind, shadcn/ui)           ← shared
Step 2:  Database schema (all entities for SaaS features)      ← SaaS
Step 3:  Auth (Clerk, middleware, protected routes)             ← shared
Step 4:  App layout (sidebar + topbar)                         ← SaaS
Step 5:  Core CRUD feature                                     ← SaaS
Step 6:  Dashboard                                             ← SaaS
Step 7:  Billing (Stripe)                                      ← SaaS
Step 8:  Settings (profile, billing, account)                  ← SaaS
Step 9:  Marketing layout (navbar + footer)                    ← Marketing
Step 10: Landing page (hero, features, testimonials, CTA)      ← Marketing
Step 11: Pricing page                                          ← Marketing
Step 12: SEO (metadata, sitemap, OG images)                    ← Marketing
Step 13: Polish (loading states, responsive, error boundaries) ← shared
Step 14: Deploy                                                ← shared
```

### Example: Mobile App + API Backend

```
Step 1:  Monorepo setup (turborepo or pnpm workspaces)        ← shared
Step 2:  API scaffolding (Hono, Drizzle, Zod)                 ← API
Step 3:  Database schema + migrations                          ← API
Step 4:  Auth API (register, login, JWT)                       ← API
Step 5:  Core feature API routes                               ← API
Step 6:  Mobile scaffolding (Expo, NativeWind)                 ← Mobile
Step 7:  Mobile auth (login, register screens)                 ← Mobile
Step 8:  Mobile core feature screens                           ← Mobile
Step 9:  Mobile data layer (TanStack Query + API client)       ← Mobile
Step 10: Push notifications                                    ← Mobile
Step 11: API: background jobs, caching                         ← API
Step 12: Polish both apps                                      ← shared
Step 13: Deploy API (Railway) + Build mobile (EAS)             ← shared
```

---

## Blueprint Modifications for Hybrids

When generating a blueprint for a hybrid project:

1. **Section 1 (Overview):** State both primary and secondary archetypes
2. **Section 2 (Tech Stack):** Unified stack — note which technologies serve which archetype
3. **Section 3 (Directory Structure):** Merged structure using route groups or monorepo
4. **Section 4 (Data Model):** All entities from both archetypes in one schema
5. **Section 9 (Build Order):** Interleaved as described above, with labels showing which archetype each step serves
6. **Section 15 (CLAUDE.md):** Single CLAUDE.md covering all parts — The Foreman includes rules from both archetypes
