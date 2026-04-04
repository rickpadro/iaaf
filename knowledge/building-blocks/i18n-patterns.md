# Building Block: Internationalization (i18n) Patterns

## Decision Matrix

| Solution | Best For | Pros | Cons | Cost |
|----------|----------|------|------|------|
| **next-intl** | Next.js App Router | Built for App Router, server components, type-safe | Next.js only | Free |
| **react-i18next** | Any React project | Largest ecosystem, flexible, plugins | More setup, not SSR-optimized | Free |
| **Paraglide** | Compile-time i18n | Zero runtime, tree-shaking, type-safe | Newer, smaller ecosystem | Free |
| **Astro i18n** | Astro projects | Built-in routing, content collections per locale | Astro only | Free |
| **Crowdin / Lokalise** | Translation management | Professional translators, CI integration, review workflows | Cost, complexity | $40/mo+ |

## Recommendation by Project Type

| Project | Recommended | Why |
|---------|-------------|-----|
| Next.js SaaS/App | **next-intl** | Best App Router integration, server component support |
| Marketing Site (Astro) | **Astro built-in** | Content collections per locale, static generation |
| React SPA | **react-i18next** | Largest ecosystem, most tutorials |
| Small project (2-3 languages) | **next-intl** with JSON files | Simple, no external service needed |
| Large project (10+ languages) | **next-intl** + **Crowdin** | Professional translation management |

## Locale Routing Strategies

| Strategy | URL Pattern | Pros | Cons |
|----------|-------------|------|------|
| **Subpath** (recommended) | `/en/about`, `/es/about` | Simple, good SEO, one deployment | URL prefix on every page |
| **Subdomain** | `en.example.com`, `es.example.com` | Clean URLs, separate analytics | DNS setup, CORS, multiple deploys |
| **Domain** | `example.com`, `example.es` | Best for regional branding | Multiple domains, complex setup |
| **Cookie/Header** | Same URL, content varies | Simple URLs | Bad for SEO, caching issues |

**Recommendation:** Always use subpath routing. It's simplest and best for SEO.

## next-intl Setup (Next.js App Router)

### Directory Structure

```
src/
  app/
    [locale]/                      # Dynamic locale segment
      page.tsx                     # Homepage
      about/page.tsx               # About page
      layout.tsx                   # Locale layout (sets lang, loads messages)
  i18n/
    request.ts                     # Server-side locale config
    routing.ts                     # Locale routing config
  messages/
    en.json                        # English translations
    es.json                        # Spanish translations
    fr.json                        # French translations
  middleware.ts                    # Locale detection + redirect
```

### Translation Files

```json
// messages/en.json
{
  "home": {
    "title": "Welcome to our platform",
    "description": "The best way to manage your projects",
    "cta": "Get Started"
  },
  "nav": {
    "home": "Home",
    "pricing": "Pricing",
    "about": "About"
  },
  "common": {
    "loading": "Loading...",
    "error": "Something went wrong",
    "save": "Save",
    "cancel": "Cancel"
  }
}
```

### Key Patterns

```typescript
// Server Component
import { useTranslations } from 'next-intl'

function HomePage() {
  const t = useTranslations('home')
  return <h1>{t('title')}</h1>
}

// With variables
// en.json: "greeting": "Hello, {name}!"
t('greeting', { name: user.name })

// With plurals
// en.json: "items": "You have {count, plural, =0 {no items} one {1 item} other {# items}}"
t('items', { count: 3 })
```

## Content Translation Strategies

| Strategy | When | How |
|----------|------|-----|
| **JSON files** (in repo) | Small app, dev-managed content | One JSON per locale, committed to git |
| **CMS per locale** | Content platform, marketing | Sanity/Contentful with locale field per document |
| **Translation service** | Large app, professional translations | Crowdin/Lokalise syncs with JSON files via CI |
| **AI translation** | MVP, budget-constrained | Translate JSON files with Claude/GPT, human review |

## Date, Number, and Currency Formatting

```typescript
// Use Intl API (built into every modern JS runtime)

// Date
new Intl.DateTimeFormat('es', { dateStyle: 'long' }).format(date)
// → "3 de abril de 2026"

// Number
new Intl.NumberFormat('de', { style: 'decimal' }).format(1234567.89)
// → "1.234.567,89"

// Currency
new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD' }).format(29.99)
// → "$29.99"

new Intl.NumberFormat('ja-JP', { style: 'currency', currency: 'JPY' }).format(2999)
// → "¥2,999"
```

**Rule:** Never format dates/numbers/currencies manually. Always use `Intl` API with the user's locale.

## RTL (Right-to-Left) Support

If supporting Arabic, Hebrew, Farsi, or Urdu:

```html
<!-- Set dir attribute based on locale -->
<html lang={locale} dir={locale === 'ar' || locale === 'he' ? 'rtl' : 'ltr'}>
```

```css
/* Use logical properties instead of physical */
/* Bad: */ margin-left: 16px;
/* Good: */ margin-inline-start: 16px;

/* Tailwind: use ms- (margin-start) and me- (margin-end) */
<div class="ms-4 me-2">  /* Works in both LTR and RTL */
```

## SEO for Multilingual Sites

```html
<!-- On every page, declare all language versions -->
<link rel="alternate" hreflang="en" href="https://example.com/en/about" />
<link rel="alternate" hreflang="es" href="https://example.com/es/about" />
<link rel="alternate" hreflang="x-default" href="https://example.com/en/about" />

<!-- Set canonical to current locale version -->
<link rel="canonical" href="https://example.com/es/about" />
```

Generate a sitemap with all locale versions of every page.

## Common Pitfalls

- **Don't concatenate strings for translations.** `"Hello" + name` breaks in languages with different word order. Use variables: `t('greeting', { name })`.
- **Don't hardcode date/number formats.** Use `Intl` API. `MM/DD/YYYY` is US-only.
- **Don't forget pluralization rules.** English has 2 forms (singular/plural). Arabic has 6. Use ICU message format.
- **Don't translate technical terms.** Keep brand names, API terms, and code identifiers untranslated.
- **Don't forget the `lang` attribute.** Screen readers and search engines need `<html lang="es">`.
- **Don't use flags for language switchers.** Flags represent countries, not languages. Use language names: "English", "Espanol".
