# Building Block: Frontend Patterns

Covers: framework selection, styling, component libraries, state management, and rendering strategies.

## Framework Decision Matrix

| Framework | Best For | Rendering | Ecosystem |
|-----------|----------|-----------|-----------|
| **Next.js 15** | Full-stack web apps, SaaS, e-commerce | SSR, SSG, ISR, Client | Largest |
| **Astro 5** | Content sites, marketing, blogs | Static + islands | Growing |
| **Nuxt 4** | Vue ecosystem | SSR, SSG, ISR | Large (Vue) |
| **SvelteKit** | Performance-critical, great DX | SSR, SSG | Smaller |
| **Vite + React** | SPAs, internal tools | Client only | Large |

**Recommendation:** Next.js 15 for apps. Astro 5 for content/marketing.

## Component Libraries

| Library | Style | Best For |
|---------|-------|----------|
| **shadcn/ui** | Radix + Tailwind, copy-paste | Any React project (default choice) |
| **Tremor** | Dashboard-focused | Data visualization, admin |
| **DaisyUI** | Tailwind plugin, themed | Fast prototyping |
| **Radix UI** | Unstyled primitives | Full custom design systems |

## Styling: Tailwind CSS v4 (Always)

### Setup (Next.js)
```css
/* globals.css */
@import "tailwindcss";

@theme {
  --color-primary: #2563eb;
  --color-secondary: #7c3aed;
  --font-sans: 'Inter', sans-serif;
}
```

### Design Token Architecture
Three layers: **Primitives** (raw values) → **Semantic** (meaning: `--color-primary`) → **Component** (usage: `--button-bg`). Define semantic tokens in `@theme`, use them everywhere.

### shadcn/ui Setup
```bash
npx shadcn@latest init
# Install first: button, input, dialog, dropdown-menu, card, table, toast, form
```

### Typography Scale
| Role | Size | Weight |
|------|------|--------|
| h1 | 36px | 700 |
| h2 | 30px | 600 |
| h3 | 24px | 600 |
| body | 16px | 400 |
| small | 14px | 400 |

### Responsive: Mobile-First
```html
<div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3">
```
Breakpoints: sm 640px, md 768px, lg 1024px, xl 1280px.

## Rendering Strategies (Next.js 15)

| Strategy | When | How |
|----------|------|-----|
| **Server Components** (default) | Data display, pages, layouts | Just fetch in the component. Zero client JS. |
| **Client Components** (`"use client"`) | Interactivity, forms, browser APIs | Keep small and leaf-level. |
| **SSG** | Marketing, blog posts | Pre-built at build time. Fastest. |
| **ISR** | Content that changes occasionally | `revalidate: 3600` — refresh hourly. |
| **SSR** | Personalized, real-time, search | `dynamic = 'force-dynamic'` |

## State Management

### The Default Stack (covers 99% of cases)

| Solution | Purpose | When |
|----------|---------|------|
| **Server Components** | Data display | Default — just fetch and render |
| **Server Actions** | Mutations (create/update/delete) | Forms, data changes |
| **TanStack Query** | Client-side server state | Polling, optimistic updates, cache across nav |
| **Zustand** | Client-only UI state | Sidebar, modals, theme, filters |

### When You DON'T Need State Management
- Data fetched by Server Components → just fetch
- Form state → React Hook Form
- URL state (filters, pagination) → `useSearchParams` + `nuqs`
- Theme → `next-themes`
- Auth state → Clerk `useAuth()` or NextAuth `useSession()`

### Real-Time (Only When Needed)

| Use Case | Solution |
|----------|----------|
| Chat, live notifications | Supabase Realtime |
| Collaborative editing | Liveblocks + Yjs |
| Live dashboards | Supabase Realtime or SSE |
| Regular dashboards | Polling every 30-60s (no real-time needed) |

## Key Patterns (Next.js 15)

### Data Fetching
```typescript
// Server Component — direct query, no API call
async function Page() {
  const data = await db.query.posts.findMany()
  return <PostList posts={data} />
}
```

### Forms
```typescript
// Server Action (recommended)
async function createPost(formData: FormData) {
  'use server'
  const data = schema.parse(Object.fromEntries(formData))
  await db.insert(posts).values(data)
  revalidatePath('/posts')
}
```

### File Structure Convention
```
app/
  (group)/           # Route group (no URL segment)
  loading.tsx        # Suspense boundary
  error.tsx          # Error boundary
  not-found.tsx      # 404
  layout.tsx         # Shared layout
  page.tsx           # Page component
```
