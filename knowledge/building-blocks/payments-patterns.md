# Building Block: Payments & Billing Patterns

## Decision Matrix

| Solution | Best For | Pros | Cons | Cost |
|----------|----------|------|------|------|
| **Stripe** | SaaS, marketplaces, complex billing | Most powerful API, excellent docs, Stripe Checkout, Billing Portal, Connect for marketplaces | Complex for simple use cases, must handle taxes yourself (or use Stripe Tax) | 2.9% + $0.30 per transaction |
| **Lemonsqueezy** | SaaS, digital products | Handles taxes/VAT globally, simpler API, acts as Merchant of Record | Less flexible than Stripe, fewer features, smaller ecosystem | 5% + $0.50 per transaction |
| **Paddle** | SaaS (B2B and B2C) | Merchant of Record (handles taxes, invoicing, compliance), good B2B features | Higher fees, less control, limited customization | 5% + $0.50 per transaction |
| **PayPal** | Consumer products, international | Ubiquitous, buyer trust, international coverage | Poor DX, complex API, disputes favor buyers | 2.9% + $0.30 per transaction |
| **Stripe + Revenue Cat** | Mobile apps | RevenueCat manages App Store/Play Store subscriptions, Stripe for web | Two systems to manage | RevenueCat free to $19/mo + Stripe fees |

## Recommendation by Project Type

| Project | Recommended | Why |
|---------|-------------|-----|
| SaaS (most cases) | **Stripe** | Most flexible, best DX, Checkout + Billing Portal save weeks of work |
| SaaS (solo founder, want simplicity) | **Lemonsqueezy** | Handles taxes, invoicing, compliance — you focus on product |
| Mobile app with subscriptions | **RevenueCat + Stripe** | RevenueCat handles App Store/Play Store, Stripe for web billing |
| Marketplace / multi-vendor | **Stripe Connect** | Built for multi-party payments, splits, payouts |
| Digital downloads | **Lemonsqueezy** or **Gumroad** | Simple, handles delivery + taxes |

## Stripe Implementation Patterns

### Subscription Flow (Most Common for SaaS)

```
User visits /pricing
    → Selects a plan
    → Stripe Checkout session created (server-side)
    → Redirected to Stripe Checkout (hosted page)
    → Pays
    → Stripe sends webhook: checkout.session.completed
    → Server updates user.subscription in database
    → User redirected to /dashboard
```

### Key Stripe Objects

| Object | Purpose |
|--------|---------|
| `Customer` | Represents a user. Create on signup, store `stripe_customer_id` in your DB |
| `Product` | What you sell (e.g., "Pro Plan"). Create in Stripe Dashboard |
| `Price` | How much it costs (e.g., $29/mo). Attached to a Product |
| `Checkout Session` | Hosted payment page. Create server-side, redirect user |
| `Subscription` | Recurring billing. Created automatically after checkout |
| `Invoice` | Generated per billing cycle. Tracks payment status |
| `Webhook` | Server-to-server notification of events |

### Webhook Handling (Critical)

```typescript
// /api/webhooks/stripe/route.ts
// MUST verify webhook signature — never trust unverified events
const event = stripe.webhooks.constructEvent(body, signature, webhookSecret)

switch (event.type) {
  case 'checkout.session.completed':
    // User just subscribed — activate their plan
    break
  case 'customer.subscription.updated':
    // Plan changed (upgrade/downgrade) — update DB
    break
  case 'customer.subscription.deleted':
    // Subscription cancelled — downgrade to free
    break
  case 'invoice.payment_failed':
    // Payment failed — notify user, maybe grace period
    break
}
```

**Webhook rules:**
- Always verify signatures with `stripe.webhooks.constructEvent()`
- Use idempotency: check if event was already processed before acting
- Respond with 200 quickly — do heavy processing async
- Handle retries: Stripe retries failed webhooks for up to 3 days
- Set up webhook endpoint in Stripe Dashboard AND in code

### Billing Portal

Stripe Billing Portal lets users manage their own subscription (upgrade, downgrade, cancel, update payment method) without you building any UI:

```typescript
const session = await stripe.billingPortal.sessions.create({
  customer: user.stripeCustomerId,
  return_url: `${APP_URL}/settings/billing`,
})
// Redirect user to session.url
```

### Database Fields for Billing

```sql
-- Add to users table
stripe_customer_id    TEXT UNIQUE
subscription_id       TEXT
subscription_status   TEXT DEFAULT 'free'  -- free, active, past_due, canceled
plan                  TEXT DEFAULT 'free'  -- free, pro, enterprise
current_period_end    TIMESTAMPTZ          -- when current billing period ends
```

## Pricing Page Patterns

### Common Pricing Structures

| Model | Example | Implementation |
|-------|---------|---------------|
| **Freemium** | Free + Pro ($29/mo) + Enterprise (custom) | Feature flags per plan, usage limits on free tier |
| **Flat-rate** | $49/mo for everything | Simple: one plan, one price |
| **Per-seat** | $10/user/month | Track team size, multiply at checkout |
| **Usage-based** | $0.01 per API call | Stripe Metered Billing, report usage via API |
| **Tiered** | 0-100 = free, 101-1000 = $29, 1000+ = $99 | Stripe tiered pricing on the Price object |

### Pricing Page Best Practices

- Show 2-3 plans maximum (more causes decision paralysis)
- Highlight the recommended plan visually
- Show annual pricing with monthly equivalent and savings percentage
- Include a "Contact us" option for enterprise
- Feature comparison table below pricing cards
- Add social proof near the CTA (customer count, testimonials)

## Common Pitfalls

- **Don't store credit card numbers.** Use Stripe Checkout or Stripe Elements. Never handle raw card data.
- **Don't skip webhook signature verification.** Anyone can POST to your webhook endpoint.
- **Don't forget to handle failed payments.** Implement a grace period + dunning emails.
- **Don't hardcode prices.** Store Stripe Price IDs in environment variables or database, not code.
- **Don't forget tax compliance.** Use Stripe Tax, Lemonsqueezy (MoR), or consult a tax professional for multi-country SaaS.
- **Don't test with real money.** Use Stripe test mode and test card numbers (4242 4242 4242 4242).

## Lemonsqueezy Implementation (Simpler Alternative)

```
User visits /pricing
    → Selects a plan
    → Redirected to Lemonsqueezy hosted checkout (with ?checkout[custom][user_id]=xxx)
    → Pays
    → Lemonsqueezy sends webhook: subscription_created
    → Server updates user subscription in database
    → User redirected to /dashboard
```

Lemonsqueezy handles: taxes, invoicing, VAT, payment methods, receipt emails. You handle: user management, feature gating, webhook processing.
