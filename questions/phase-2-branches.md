# Phase 2: Archetype-Specific Deep Dive

After classifying the project in Phase 1, use the matching section below. Ask 3-5 questions — adapt based on what the user already told you. Skip questions they've already answered.

---

## SaaS Web App

### Q1: Core Data Model
"What's the main thing users create or manage in your app? (e.g., projects, invoices, tasks, courses)"

### Q2: Auth & Users
"Do users need accounts? Teams/organizations? Role-based permissions (admin, member, viewer)?"

### Q3: Payments
"Is this monetized? Free, freemium, paid subscriptions, one-time purchase, or usage-based?"

### Q4: Real-Time
"Does anything need to update in real-time? (chat, notifications, live collaboration, dashboards)"

### Q5: Integrations
"Any third-party integrations? (email sending, calendar, Stripe, Slack, external APIs)"

**Building blocks to load based on answers:**
- If auth needed → `knowledge/building-blocks/auth-patterns.md`
- If payments → load Stripe patterns from archetype
- If real-time → `knowledge/building-blocks/frontend-patterns.md` (real-time section)
- Always load → `knowledge/building-blocks/database-patterns.md`

---

## Marketing Site

### Q1: Pages
"How many pages? Just a single landing page, or multiple pages (about, pricing, features, blog)?"

### Q2: Content Management
"Will you update content frequently? Need a CMS, or is static content fine?"

### Q3: Lead Capture
"Do you need forms? (contact, newsletter signup, waitlist, demo request)"

### Q4: SEO Priority
"How important is SEO? Need to rank for specific keywords?"

### Q5: Multilingual
"Single language or multilingual?"

**Building blocks to load:**
- If CMS needed → `knowledge/building-blocks/frontend-patterns.md` (Astro/Next.js comparison)
- If SEO priority → note `/seo-audit` skill for build phase
- Always load → `knowledge/building-blocks/deployment-patterns.md`

---

## Mobile App

### Q1: Platforms
"iOS only, Android only, or both?"

### Q2: Framework
"Any preference? React Native, Flutter, or native (Swift/Kotlin)?"

### Q3: Offline
"Does the app need to work offline? How critical is offline access?"

### Q4: Device Features
"Do you need access to camera, GPS, Bluetooth, push notifications, or other device features?"

### Q5: Backend
"Does this connect to an existing API, or do we need to design the backend too?"

**Building blocks to load:**
- If needs backend → `knowledge/building-blocks/api-design-patterns.md`
- Always load → `knowledge/building-blocks/auth-patterns.md` (mobile auth patterns)
- If offline → `knowledge/building-blocks/frontend-patterns.md` (state management section)

---

## API / Backend Service

### Q1: API Style
"REST, GraphQL, or tRPC? Any preference?"

### Q2: Consumers
"Who consumes this API? Your own frontend, mobile app, third-party developers, or all of the above?"

### Q3: Auth Method
"How do consumers authenticate? API keys, JWT tokens, OAuth, or session-based?"

### Q4: Background Work
"Are there background jobs? (email sending, file processing, scheduled tasks, webhooks)"

### Q5: Scale & Performance
"Expected request volume? Any caching or rate limiting needs?"

**Building blocks to load:**
- Always → `knowledge/building-blocks/api-design-patterns.md`
- Always → `knowledge/building-blocks/database-patterns.md`
- If background jobs → `knowledge/building-blocks/deployment-patterns.md`
- If auth → `knowledge/building-blocks/auth-patterns.md`

---

## Internal Tool

### Q1: Data Source
"What data does this tool work with? Existing database, spreadsheets, external APIs?"

### Q2: User Count
"How many people will use this? Just you, a small team, or a whole department?"

### Q3: Core Actions
"What do users mainly do? View dashboards, fill forms, approve workflows, manage records?"

### Q4: Access Control
"Does everyone see everything, or do you need roles (admin, editor, viewer)?"

### Q5: Reporting
"Do you need exports (CSV, PDF) or reporting/analytics features?"

**Building blocks to load:**
- If existing database → `knowledge/building-blocks/database-patterns.md`
- If roles → `knowledge/building-blocks/auth-patterns.md`
- Always → `knowledge/building-blocks/frontend-patterns.md`

---

## Content Platform

### Q1: Content Type
"What kind of content? Blog posts, documentation, courses, user-generated content, multimedia?"

### Q2: Content Source
"Who creates content? Editorial team, users, or both?"

### Q3: Discovery
"How do users find content? Search, categories, recommendations, feeds?"

### Q4: Social Features
"Do you need comments, likes, sharing, user profiles, or following?"

### Q5: Monetization
"How is this monetized? Ads, subscriptions, gated content, sponsorships, or free?"

**Building blocks to load:**
- If user-generated → `knowledge/building-blocks/auth-patterns.md`
- If search → `knowledge/building-blocks/database-patterns.md` (full-text search)
- If monetization → load payment patterns
- Always → `knowledge/building-blocks/frontend-patterns.md`

---

## E-Commerce Store

### Q1: Product Type
"What are you selling? Physical products, digital downloads, subscriptions, or a mix?"

### Q2: Catalog Size
"How many products? Under 50, hundreds, or thousands? Do products have variants (size, color)?"

### Q3: Commerce Engine
"Any preference for the commerce backend? Shopify (managed), Medusa (open-source headless), or building custom?"

### Q4: Payments & Regions
"Which countries do you sell to? Do you need multi-currency, tax handling, or specific payment methods?"

### Q5: Inventory & Fulfillment
"Do you need inventory tracking, shipping rate calculation, or integration with fulfillment services?"

**Building blocks to load:**
- Always → `knowledge/building-blocks/payments-patterns.md`
- Always → `knowledge/building-blocks/integrations-patterns.md` (product images)
- Always → `knowledge/building-blocks/security-ops-patterns.md`
- If digital products → `knowledge/building-blocks/integrations-patterns.md` (delivery)

---

## AI / ML Application

### Q1: AI Type
"What kind of AI feature? Chat/conversational, document Q&A (RAG), content generation, code assistant, image generation, or something else?"

### Q2: LLM Provider
"Any preference for the AI model? Anthropic (Claude), OpenAI (GPT), Google (Gemini), or open-source (Llama, Mistral)?"

### Q3: Data Sources
"Does the AI need access to specific data? User-uploaded documents, a knowledge base, a database, or external APIs?"

### Q4: Usage & Cost
"How do you plan to handle AI costs? Pass through to users (usage-based pricing), absorb the cost, or set usage limits?"

### Q5: Privacy
"How sensitive is the data being sent to the AI? Any restrictions on sending data to external APIs?"

**Building blocks to load:**
- If documents → `knowledge/building-blocks/integrations-patterns.md`
- If billing → `knowledge/building-blocks/payments-patterns.md`
- Always → `knowledge/building-blocks/api-design-patterns.md`
- Always → `knowledge/building-blocks/auth-patterns.md`

---

## CLI Tool

### Q1: Purpose
"What does the CLI do? Code generation, project scaffolding, API interaction, file processing, automation?"

### Q2: Language
"Preference for the CLI language? Node.js/TypeScript, Go, Rust, or Python?"

### Q3: Interactivity
"Is it interactive (prompts, menus) or purely flag-driven (non-interactive, scriptable)?"

### Q4: Configuration
"Does it need a config file? Per-project, global, or both?"

### Q5: Distribution
"How will users install it? npm, Homebrew, binary download, or pip?"

**Building blocks to load:**
- Always → `knowledge/building-blocks/testing-patterns.md`

---

## Real-Time / Collaborative App

### Q1: Collaboration Type
"What do users collaborate on? Documents, whiteboards, code, designs, spreadsheets, or something else?"

### Q2: Concurrency
"How many users will edit simultaneously? 2-5 (small team), 10-50 (large team), or 100+ (public)?"

### Q3: Offline Support
"Does the app need to work offline and sync later?"

### Q4: Real-Time Features
"Which real-time features do you need? Live cursors, presence (who's online), collaborative editing, chat, notifications?"

### Q5: Persistence
"Does the collaborative state need to be saved permanently, or is it session-only (like a call)?"

**Building blocks to load:**
- Always → `knowledge/building-blocks/frontend-patterns.md`
- Always → `knowledge/building-blocks/auth-patterns.md`
- If offline → `knowledge/building-blocks/database-patterns.md`
- Always → `knowledge/building-blocks/frontend-patterns.md`
