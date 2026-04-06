# Reference: Image Optimization Patterns

> Supplementary detail for performance-patterns.md. Load only when optimizing images.

## Sizing Rules

| Context | Max Width | Format | Loading |
|---------|-----------|--------|---------|
| Hero / banner | 1200-1600px | AVIF with WebP fallback | `priority` (eager) |
| Card thumbnail | 400-600px | WebP | `lazy` |
| Avatar | 128-256px | WebP | `lazy` |
| Icon / logo | 64-128px | SVG (vector) or WebP | Inline or eager |
| Product image | 600-800px | WebP | `lazy` with `sizes` |
| Blog content | 800-1200px | WebP | `lazy` |

## Next.js Image

```tsx
// Hero — loads immediately
<Image src="/hero.png" alt="Product" width={1200} height={675} priority sizes="100vw" />

// Card — lazy loaded, responsive
<Image
  src={product.imageUrl}
  alt={product.name}
  width={400}
  height={300}
  sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
/>
```

## Astro Image

```astro
---
import { Image } from 'astro:assets'
import heroImage from '../assets/hero.png'
---
<Image src={heroImage} alt="Hero" width={1200} format="avif" />
```

## Cloudinary URL Transforms

```
Original:  https://res.cloudinary.com/demo/image/upload/sample.jpg
Thumbnail: https://res.cloudinary.com/demo/image/upload/w_200,h_200,c_fill/sample.jpg
WebP auto: https://res.cloudinary.com/demo/image/upload/f_auto,q_auto,w_600/sample.jpg
```

## Self-Hosted Font Optimization

```typescript
// Next.js — auto self-hosts, prevents layout shift
import { Inter, JetBrains_Mono } from 'next/font/google'
const inter = Inter({ subsets: ['latin'], display: 'swap', variable: '--font-sans' })
```

```css
/* Manual self-hosting (Astro, plain HTML) */
@font-face {
  font-family: 'Inter';
  src: url('/fonts/inter-var.woff2') format('woff2');
  font-weight: 100 900;
  font-display: swap;
}
```

## Bundle Analysis

```bash
# Next.js
ANALYZE=true pnpm build

# Check package impact before installing
# Visit bundlephobia.com/package/{name}
```

Rules:
- No package over 50KB gzipped without justification
- Prefer ESM packages (tree-shakeable)
- Use native alternatives when possible (lodash → es-toolkit)
