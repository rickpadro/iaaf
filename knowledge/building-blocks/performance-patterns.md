# Building Block: Performance Patterns

## Core Web Vitals

| Metric | What It Measures | Good | Needs Work | Poor |
|--------|-----------------|------|------------|------|
| **LCP** (Largest Contentful Paint) | Loading speed — when main content is visible | < 2.5s | 2.5-4s | > 4s |
| **INP** (Interaction to Next Paint) | Responsiveness — delay after user interaction | < 200ms | 200-500ms | > 500ms |
| **CLS** (Cumulative Layout Shift) | Visual stability — does content jump around? | < 0.1 | 0.1-0.25 | > 0.25 |

### How to Optimize Each

**LCP — Make the main content load fast:**
- Optimize the hero image (WebP/AVIF, proper sizing, preload)
- Use Server Components (HTML arrives with data, no client fetch)
- Self-host fonts (no Google Fonts network request)
- Preload critical resources: `<link rel="preload">`

**INP — Make interactions respond instantly:**
- Keep Client Components small and leaf-level
- Use `startTransition` for non-urgent state updates
- Avoid blocking the main thread (no heavy computation in event handlers)
- Use Web Workers for CPU-intensive tasks

**CLS — Don't shift content around:**
- Always set `width` and `height` on images and videos
- Use `aspect-ratio` CSS for responsive media
- Reserve space for dynamic content (ads, embeds, async-loaded items)
- Use `font-display: swap` with proper fallback font sizing

## Image Optimization

### Strategy by Framework

| Framework | Tool | How |
|-----------|------|-----|
| Next.js | `next/image` | `<Image>` auto-optimizes, resizes, converts to WebP, lazy loads |
| Astro | `astro:assets` | `<Image>` component optimizes at build time |
| Manual | Cloudinary / Sharp | URL-based transforms or build-time processing |

### Image Sizing Rules

| Context | Max Width | Format | Loading |
|---------|-----------|--------|---------|
| Hero / banner | 1200-1600px | AVIF with WebP fallback | `priority` (eager) |
| Card thumbnail | 400-600px | WebP | `lazy` |
| Avatar | 128-256px | WebP | `lazy` |
| Icon / logo | 64-128px | SVG (vector) or WebP | Inline or eager |

### Next.js Image Best Practices

```tsx
// Hero image — loads immediately, priority
<Image
  src="/hero.png"
  alt="Product screenshot"
  width={1200}
  height={675}
  priority
  sizes="100vw"
/>

// Card image — lazy loaded, responsive
<Image
  src={product.imageUrl}
  alt={product.name}
  width={400}
  height={300}
  sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
/>
```

## Bundle Optimization

### Code Splitting

```typescript
// Dynamic import — only loads when needed
const HeavyChart = dynamic(() => import('@/components/HeavyChart'), {
  loading: () => <ChartSkeleton />,
  ssr: false,  // Don't render on server if it's client-only
})

// Route-based splitting — Next.js does this automatically per page
// Each page is its own bundle, loaded on navigation
```

### Tree Shaking

```typescript
// BAD — imports entire library
import _ from 'lodash'
_.debounce(fn, 300)

// GOOD — imports only the function
import debounce from 'lodash/debounce'
debounce(fn, 300)

// BEST — use native or smaller alternative
// Native: setTimeout + clearTimeout
// Or: import { debounce } from 'es-toolkit'  (tiny, tree-shakeable)
```

### Analyzing Bundle Size

```bash
# Next.js: see bundle analysis
ANALYZE=true pnpm build

# Or use: npx @next/bundle-analyzer
```

**Rules:**
- No package over 50KB gzipped without justification
- Prefer packages with ESM exports (tree-shakeable)
- Check bundle impact before adding a dependency: `bundlephobia.com`

## Caching Strategy

### Browser Cache

```typescript
// Static assets (images, fonts, JS/CSS with hash): cache forever
Cache-Control: public, max-age=31536000, immutable

// HTML pages: always revalidate
Cache-Control: public, max-age=0, must-revalidate

// API responses: short cache
Cache-Control: public, max-age=60, stale-while-revalidate=300
```

### Next.js Caching Layers

| Layer | What | Default | Override |
|-------|------|---------|---------|
| **Data Cache** | fetch() results | Cached indefinitely | `{ next: { revalidate: 3600 } }` |
| **Full Route Cache** | Static pages | Cached at build | `revalidatePath()` or `revalidateTag()` |
| **Router Cache** | Client-side navigation | 30s dynamic, 5min static | `router.refresh()` |

### ISR (Incremental Static Regeneration)

```typescript
// Page revalidates every hour
export const revalidate = 3600

// Or revalidate on demand (webhook from CMS)
// api/revalidate/route.ts
export async function POST(req: Request) {
  const { path } = await req.json()
  revalidatePath(path)
  return Response.json({ revalidated: true })
}
```

## Database Query Optimization

### Indexing Rules

```sql
-- Index all foreign keys
CREATE INDEX idx_tasks_user_id ON tasks(user_id);
CREATE INDEX idx_tasks_list_id ON tasks(list_id);

-- Index columns used in WHERE clauses
CREATE INDEX idx_tasks_status ON tasks(status);

-- Composite index for common query patterns
CREATE INDEX idx_tasks_user_status ON tasks(user_id, status);

-- Partial index for common filters
CREATE INDEX idx_tasks_active ON tasks(user_id) WHERE deleted_at IS NULL;
```

### Connection Pooling

```
Without pooling: each request opens a new DB connection (slow, limited)
With pooling: connections are reused across requests (fast, scalable)

Supabase: built-in pooler (use the pooler URL, port 6543)
Neon: built-in pooler
Self-hosted: PgBouncer
```

### N+1 Query Prevention

```typescript
// BAD — N+1: one query per task to get the assignee
const tasks = await db.query.tasks.findMany()
for (const task of tasks) {
  task.assignee = await db.query.users.findFirst({ where: eq(users.id, task.assigneeId) })
}

// GOOD — eager load with join
const tasks = await db.query.tasks.findMany({
  with: { assignee: true }  // Prisma/Drizzle eager loading
})
```

## Font Optimization

```typescript
// Next.js: use next/font (auto self-hosts, no layout shift)
import { Inter } from 'next/font/google'
const inter = Inter({ subsets: ['latin'], display: 'swap', variable: '--font-sans' })

// Astro: self-host manually
// 1. Download .woff2 files to public/fonts/
// 2. Declare @font-face in CSS
// 3. Set font-display: swap
```

**Rules:**
- Self-host fonts (don't load from Google Fonts CDN)
- Use `font-display: swap` to prevent invisible text
- Load only the weights and subsets you use
- Use variable fonts when possible (one file for all weights)

## Common Pitfalls

- **Don't optimize prematurely.** Ship first, measure, then optimize the bottlenecks. Lighthouse and real user metrics tell you where to focus.
- **Don't lazy load above-the-fold content.** Images and components visible on initial load should be eager/priority.
- **Don't use `use client` on everything.** Every Client Component adds to the JS bundle. Server Components send zero JS.
- **Don't skip `sizes` on responsive images.** Without `sizes`, the browser downloads the largest image for all viewports.
- **Don't add heavy animation libraries for simple transitions.** CSS transitions and `@keyframes` handle 90% of cases. Only add Framer Motion if you need gesture-based or complex animations.
