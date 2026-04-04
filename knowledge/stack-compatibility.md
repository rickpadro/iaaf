# Stack Compatibility Matrix

Use this to validate tech stack decisions. Avoid known bad combinations.

## Proven Combinations (Recommended)

### The Modern SaaS Stack
```
Next.js 15 + TypeScript + Tailwind v4 + shadcn/ui + Supabase + Clerk + Stripe + Vercel
```
Why it works: Everything integrates seamlessly. Largest ecosystem. Most tutorials/examples.

### The Lightweight SaaS Stack
```
Next.js 15 + TypeScript + Tailwind v4 + shadcn/ui + Supabase (Auth + DB) + Lemonsqueezy + Vercel
```
Why it works: Fewer services. Supabase handles auth AND database. Lemonsqueezy handles taxes.

### The API-First Stack
```
Hono + TypeScript + Drizzle + PostgreSQL (Neon) + Railway
```
Why it works: Lightweight, fast, SQL-close. Hono runs everywhere (Node, Bun, Cloudflare Workers).

### The Content Stack
```
Astro 5 + TypeScript + Tailwind v4 + Sanity + Vercel
```
Why it works: Zero JS by default. Sanity's content lake is powerful. Fast sites.

### The Mobile Stack
```
Expo (React Native) + TypeScript + NativeWind + Supabase + EAS Build
```
Why it works: Expo simplifies everything. NativeWind brings Tailwind to mobile. Supabase covers backend.

### The Internal Tool Stack
```
Next.js 15 + TypeScript + Tailwind v4 + shadcn/ui + Prisma + PostgreSQL + Vercel
```
Why it works: Fast to build. shadcn/ui has all the admin components (tables, forms, dialogs).

### The E-Commerce Stack
```
Next.js 15 + TypeScript + Tailwind v4 + shadcn/ui + Medusa.js 2.0 + Stripe + Cloudinary + Vercel (frontend) + Railway (Medusa)
```
Why it works: Medusa handles commerce logic (products, cart, orders, inventory). Next.js for a fast storefront. Cloudinary for image optimization at scale.

### The AI App Stack
```
Next.js 15 + TypeScript + Tailwind v4 + shadcn/ui + Vercel AI SDK + Anthropic/OpenAI + Pinecone + Supabase + Vercel
```
Why it works: Vercel AI SDK handles streaming. Pinecone for vector search. Supabase for user data + pgvector as alternative to Pinecone.

### The CLI Stack
```
Node.js + TypeScript + Commander.js + @inquirer/prompts + chalk + ora + tsup
```
Why it works: Commander.js is the standard. Inquirer for interactive prompts. tsup bundles to a single file. Publish to npm.

### The Real-Time Collaboration Stack
```
Next.js 15 + TypeScript + Tailwind v4 + shadcn/ui + Liveblocks + Yjs + Tiptap + Clerk + Vercel
```
Why it works: Liveblocks handles real-time sync, presence, and storage. Yjs (CRDT) for conflict-free collaborative editing. Tiptap as the rich text editor.

## Known Bad Combinations (Avoid)

| Combination | Problem |
|-------------|---------|
| Tailwind + Styled Components | Conflicting paradigms. Pick one. |
| Prisma + Cloudflare Workers | Prisma doesn't run on Workers edge runtime. Use Drizzle. |
| NextAuth + Clerk | Both do auth. Use one. |
| GraphQL + simple CRUD app | Over-engineering. REST or tRPC is simpler. |
| MongoDB + relational data | If you have joins and foreign keys, use PostgreSQL. |
| Socket.io + Vercel | Vercel is serverless. WebSockets need persistent connections. Use Supabase Realtime or Pusher. |
| Redux + small app | Overkill. Zustand or just useState is fine. |
| Firebase + Prisma | Firebase uses Firestore (NoSQL). Prisma is for SQL databases. |
| Next.js + Express | Next.js has built-in API routes. Adding Express is redundant. |
| Tailwind v3 patterns in v4 | v4 changed config format. Don't mix `tailwind.config.js` with `@theme`. |

## Compatibility Notes

### Auth + Database Pairings
| Auth | Best DB Partner | Why |
|------|-----------------|-----|
| Clerk | Any (Prisma/Drizzle + any DB) | Clerk is independent — stores users separately, sync via webhooks |
| Supabase Auth | Supabase (Postgres) | Tight integration, RLS policies, same dashboard |
| NextAuth | Any (with adapter) | Flexible — Prisma adapter, Drizzle adapter, etc. |
| Firebase Auth | Firestore | Same ecosystem, seamless integration |

### Hosting + Framework Pairings
| Framework | Best Host | Why |
|-----------|-----------|-----|
| Next.js | Vercel | Built by same team, zero-config, edge middleware |
| Astro | Vercel or Cloudflare Pages | Both excellent for static + SSR |
| Hono / Express | Railway or Fly.io | Need persistent Node.js process |
| React Native | EAS Build (Expo) | Integrated build + OTA updates |

### ORM + Database Pairings
| ORM | Best Database | Notes |
|-----|--------------|-------|
| Prisma | PostgreSQL (Supabase/Neon) | Best DX, great migrations |
| Drizzle | PostgreSQL or SQLite (Turso) | Performance, SQL-like, edge compatible |
| Mongoose | MongoDB | Only ORM for MongoDB |
