# Archetype: E-Commerce Store

## Default Stack Recommendation

| Layer | Default | Alternative | When to Switch |
|-------|---------|-------------|----------------|
| Framework | Next.js 15 (App Router) | Astro 5 + React islands | Simple catalog, no dynamic features |
| Language | TypeScript (strict) | — | Never compromise on this |
| Styling | Tailwind CSS v4 | — | Always Tailwind |
| Components | shadcn/ui | Radix + custom | Custom design system |
| Commerce Engine | Medusa.js 2.0 | Shopify Storefront API | Shopify if already using Shopify, or want managed infrastructure |
| Database | PostgreSQL (Supabase/Neon) | — | Medusa includes its own DB, or use Supabase for custom |
| ORM | Prisma | Drizzle | If not using Medusa (which has its own data layer) |
| Auth | Medusa Auth / Clerk | NextAuth | Clerk for advanced social login, NextAuth for budget |
| Payments | Stripe | Lemonsqueezy, PayPal | Lemonsqueezy for digital products, PayPal for international |
| Search | Algolia | Meilisearch (self-hosted) | Meilisearch if budget-constrained |
| Media/Images | Cloudinary | Uploadthing, Supabase Storage | Cloudinary for heavy image transforms |
| Email | Resend | SendGrid | SendGrid for marketing + transactional |
| Hosting | Vercel (frontend) + Railway (Medusa) | Vercel only (if Shopify API) | Shopify handles backend if using Storefront API |
| Package Manager | pnpm | npm | Simplicity |

## Default Directory Structure

### Headless with Medusa.js

```
storefront/                        # Next.js frontend
  src/
    app/
      (shop)/
        page.tsx                   # Homepage — featured products, hero, categories
        products/
          page.tsx                 # Product listing with filters/search
          [handle]/page.tsx        # Product detail page
        categories/
          [slug]/page.tsx          # Category page
        cart/page.tsx              # Cart page
        checkout/page.tsx          # Checkout flow
      (account)/
        login/page.tsx             # Customer login
        register/page.tsx          # Customer registration
        orders/page.tsx            # Order history
        orders/[id]/page.tsx       # Order detail
        profile/page.tsx           # Account settings
      api/
        webhooks/
          stripe/route.ts          # Payment webhook
          medusa/route.ts          # Medusa event webhook
      layout.tsx
      globals.css
    components/
      ui/                          # shadcn/ui primitives
      shop/
        ProductCard.tsx            # Product card for listings
        ProductGallery.tsx         # Image gallery with zoom
        AddToCartButton.tsx        # Add to cart with variant selection
        PriceDisplay.tsx           # Price with currency, discounts
        QuantitySelector.tsx       # +/- quantity input
      cart/
        CartDrawer.tsx             # Slide-out cart
        CartItem.tsx               # Individual cart item
        CartSummary.tsx            # Subtotal, shipping, tax, total
      checkout/
        CheckoutForm.tsx           # Shipping + payment form
        OrderSummary.tsx           # Order review before payment
      layout/
        Header.tsx                 # Nav with cart icon + count
        Footer.tsx                 # Links, newsletter signup
        SearchBar.tsx              # Product search with instant results
    lib/
      medusa/
        client.ts                  # Medusa JS client
        actions.ts                 # Server Actions for cart, checkout
      stripe/
        client.ts
      utils.ts
    types/
      index.ts
  public/
    logo.svg
    og-image.png

backend/                           # Medusa.js backend (if self-hosted)
  src/
    api/                           # Custom API routes
    subscribers/                   # Event handlers
    services/                      # Custom business logic
    models/                        # Custom data models
  medusa-config.ts
```

### With Shopify Storefront API (Simpler)

```
src/
  app/
    (shop)/                        # Same frontend structure as above
    api/
      revalidate/route.ts          # Shopify webhook to revalidate pages
  lib/
    shopify/
      client.ts                    # Shopify Storefront API client (GraphQL)
      queries.ts                   # Product, collection, cart queries
      mutations.ts                 # Cart mutations, checkout
    utils.ts
```

## Common Patterns

### Product Display
- Product listing with grid/list view toggle
- Filters: category, price range, size, color, rating
- Sort: price low/high, newest, bestselling, rating
- Pagination or infinite scroll
- Quick view modal from listing

### Cart
- Cart persisted in cookies/localStorage (guest) or database (authenticated)
- Slide-out drawer (not a separate page) for quick add/continue shopping
- Real-time stock validation on add-to-cart
- Quantity update and remove directly in cart

### Checkout Flow
```
Cart
  → Shipping address (form with validation)
  → Shipping method selection
  → Payment (Stripe Elements or Stripe Checkout)
  → Order confirmation page
  → Confirmation email sent
```

### Product Variants
- Color, size, material → variant combinations
- Each variant has its own: price, stock count, SKU, image
- Variant selector updates price and image dynamically

### Inventory
- Track stock per variant
- Show "In stock", "Low stock (3 left)", "Out of stock"
- Prevent adding out-of-stock items to cart
- Optional: backorder / waitlist for out-of-stock

## SEO (Critical for E-Commerce)

- **Product pages:** title, description, price, availability in meta + JSON-LD (Product schema)
- **Category pages:** proper H1, description, canonical URL
- **Sitemap:** dynamic sitemap with all product and category URLs
- **Image alt text:** descriptive, includes product name
- **URL structure:** `/products/blue-running-shoes` (human-readable slugs)
- **Breadcrumbs:** with JSON-LD BreadcrumbList schema

## Build Order

1. **Scaffolding**: Next.js, Tailwind, shadcn/ui. Set up Medusa backend (or Shopify API client).
2. **Data model**: Products, categories, variants. Seed sample data.
3. **Product listing**: Grid page with basic cards, category filtering.
4. **Product detail**: Image gallery, variant selector, price, add-to-cart button.
5. **Cart**: Cart drawer, add/remove/update quantity, persist in cookies.
6. **Search**: Algolia/Meilisearch integration, search bar with instant results.
7. **Auth**: Customer login/register, account pages.
8. **Checkout**: Shipping form, payment integration (Stripe), order creation.
9. **Order management**: Order confirmation page, order history, order detail.
10. **Homepage**: Hero, featured products, categories, promotions.
11. **SEO**: Metadata, JSON-LD, sitemap, OG images per product.
12. **Email**: Order confirmation, shipping notification (Resend).
13. **Polish**: Loading states, empty states, responsive design, image optimization.
14. **Deploy**: Vercel (frontend) + Railway (Medusa backend). Custom domain, SSL.

## Common Pitfalls

- **Don't build a custom commerce engine.** Use Medusa, Shopify, or Saleor. Commerce has too many edge cases (tax, inventory, returns, shipping zones).
- **Don't skip image optimization.** Product images are the #1 page weight. Use Cloudinary or Next.js Image with aggressive optimization.
- **Don't forget mobile.** 60%+ of e-commerce traffic is mobile. Design mobile-first.
- **Don't hardcode prices.** Currency, discounts, and tax must be dynamic.
- **Don't skip structured data.** Google Shopping and rich results require Product schema JSON-LD.
- **Don't ignore page speed.** Every 100ms of latency costs ~1% in conversion. Optimize aggressively.

## Skills for Build Phase

| Skill | When |
|-------|------|
| `/frontend-design` | Product pages, checkout, homepage |
| `/shadcn-ui` | Component setup |
| `/ui-ux-pro-max` | Design system, product gallery style |
| `/seo-audit` | After product pages are built |

## See Also

- `knowledge/building-blocks/payments-patterns.md` — Stripe checkout, webhooks, subscription patterns
- `knowledge/building-blocks/integrations-patterns.md` — Product image storage, CDN, optimization
- `knowledge/building-blocks/frontend-patterns.md` — Next.js rendering strategies for commerce
- `knowledge/building-blocks/deployment-patterns.md` — Vercel + Railway, CI/CD for headless commerce
- `knowledge/building-blocks/security-ops-patterns.md` — Payment security, input validation, monitoring
