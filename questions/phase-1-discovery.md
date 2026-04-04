# Phase 1: Discovery Questions

These questions identify WHAT the user wants to build and classify it into an archetype. Ask conversationally — 2-3 questions at a time, not all at once.

---

## Core Questions

### Q1: The Vision
**Ask:** "What are you building? Describe it in your own words — what does it do, what problem does it solve?"

**Why:** This is open-ended on purpose. Let the user paint the picture. Listen for keywords that signal the archetype.

**Archetype signals:**
- "users sign up", "subscription", "dashboard", "SaaS" → **saas-webapp**
- "landing page", "marketing", "convert", "launch" → **marketing-site**
- "iOS", "Android", "mobile", "app store" → **mobile-app**
- "API", "endpoints", "microservice", "backend" → **api-backend**
- "admin panel", "internal", "dashboard for our team" → **internal-tool**
- "blog", "content", "articles", "documentation" → **content-platform**

### Q2: The Audience
**Ask:** "Who will use this? End users? Your internal team? Other developers? How many users are you expecting?"

**Why:** Determines scale, auth complexity, and UX expectations. A tool for 5 internal users is fundamentally different from a public SaaS.

### Q3: The Stage
**Ask:** "What stage is this? Quick prototype to validate an idea, or production-ready from day one?"

**Why:** MVP means smart defaults and speed. Production means error handling, testing, monitoring, CI/CD.

### Q4: Tech Preferences (if not already clear)
**Ask:** "Do you have any tech preferences or constraints? Existing stack, favorite framework, hosting requirement?"

**Why:** Some users have strong opinions (must use Vue, must deploy on AWS). Others want a recommendation. Adapt accordingly.

### Q5: Timeline (optional — ask only if relevant)
**Ask:** "What's the timeline? Weekend project or multi-week build?"

**Why:** Affects scope of the blueprint. Weekend project = minimal viable set. Multi-week = full architecture.

### Q6: Budget & Services (optional — ask if relevant)
**Ask:** "Are you open to paid services (Clerk for auth, Vercel Pro for hosting), or do you prefer free/open-source alternatives?"

**Why:** Determines whether to recommend paid services (Clerk, Vercel Pro, Algolia) or free alternatives (NextAuth, Railway free tier, Meilisearch). A solo developer with no budget needs a fundamentally different stack than a funded startup.

**How answers affect decisions:**
- "Free only" → NextAuth (not Clerk), Supabase free tier, self-hosted search, Railway free
- "Some budget" → Clerk free tier, Vercel hobby, Supabase free tier
- "Budget is not an issue" → Best tool for the job regardless of cost

### Q7: Team Experience (optional — ask if relevant)
**Ask:** "Who's building this after the blueprint is generated? Are they comfortable with the recommended stack, or are they learning?"

**Why:** A junior developer or someone new to the stack needs simpler tools, more guardrails, and a more detailed build order. A senior team can handle more complex architectures and less hand-holding.

**How answers affect decisions:**
- Junior / learning → simpler stack, more comments in blueprint, smaller steps in build order, avoid advanced patterns (tRPC, Server Actions)
- Senior / experienced → optimal stack, concise build order, can use advanced patterns
- Solo developer → minimize number of services, prefer integrated solutions (Supabase for auth + DB + storage)

### Q8: Compliance (optional — ask only if user mentions sensitive data, healthcare, finance, or EU users)
**Ask:** "Any regulatory or compliance requirements? GDPR, HIPAA, data residency, SOC2?"

**Why:** Compliance requirements fundamentally change architecture decisions. They affect where data is hosted, how it's encrypted, what audit logging is needed, which auth provider to use, and whether you need data deletion workflows.

**How answers affect decisions:**
- GDPR → data residency in EU, cookie consent, right to deletion, privacy policy, DPA with all services
- HIPAA → BAA with hosting/database provider, encryption at rest, audit logging, access controls
- SOC2 → audit logging, access controls, incident response plan, vendor assessment
- No compliance → proceed normally

---

## Classification Logic

After Q1-Q3, you should be able to classify. If ambiguous, ask one clarifying question rather than guessing.

| If the project is primarily... | Archetype |
|-------------------------------|-----------|
| A web app where users create accounts and use features | `saas-webapp` |
| A static or semi-static site to present/sell something | `marketing-site` |
| A native or cross-platform mobile app | `mobile-app` |
| A headless API consumed by other services/apps | `api-backend` |
| A tool used internally by a team (not public-facing) | `internal-tool` |
| Centered around creating, organizing, or displaying content | `content-platform` |
| An online store selling physical or digital products | `e-commerce` |
| An application powered by AI/LLM features (chat, RAG, generation) | `ai-ml-app` |
| A command-line tool or terminal utility | `cli-tool` |
| A collaborative app where multiple users interact in real-time | `realtime-collab` |

**Hybrid projects:** If a project spans archetypes (e.g., SaaS with a marketing landing page), use the PRIMARY archetype and note the secondary. Read `knowledge/hybrid-patterns.md` for how to merge architectures, directory structures, and build orders.

---

## After Classification

1. Tell the user which archetype you've identified and why
2. Read `knowledge/archetypes/<archetype>.md`
3. Proceed to Phase 2
