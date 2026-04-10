# Blueprint Completeness Checklist

Use this checklist during Phase 4 (Generate) before saving the blueprint. Verify every applicable item is covered. Do not save a blueprint that fails required checks.

---

## Required for ALL Blueprints

- [ ] All 16 sections present and filled (no empty placeholder sections)
- [ ] Build order (Section 9) has numbered steps with specific deliverables per step
- [ ] Build order includes Build Efficiency Guidelines preamble
- [ ] Every technology in Tech Stack (Section 2) has a rationale in the "Why" column
- [ ] Directory structure (Section 3) matches the chosen tech stack and framework
- [ ] Environment variables (Section 10) listed for every external service in the stack
- [ ] Dependencies (Section 11) list every package with its purpose
- [ ] CLAUDE.md (Section 15) is under 120 lines and includes concrete values (hex codes, px sizes)
- [ ] CLAUDE.md includes The Foreman section with universal rules + archetype-specific additions
- [ ] CLAUDE.md includes Tool Call Awareness section
- [ ] Non-Negotiable Rules (Section 16) has at least 5 rules
- [ ] Architecture Diagrams (Section 4b) has at least system architecture + data model diagrams
- [ ] Blueprint is written in the user's language (detected in Phase 1)

## Required for Projects WITH a Frontend

- [ ] Design System (Section 7) has hex color values for all roles (primary, secondary, background, surface, text, muted, destructive)
- [ ] Typography has specific font families, sizes in px, and weights
- [ ] Component hierarchy shown for at least 2 key pages
- [ ] Responsive breakpoints defined
- [ ] Spacing scale defined with concrete values
- [ ] Border radius values specified

## Required for Projects WITH a Database

- [ ] All entities documented with field names, types, and notes (Section 4)
- [ ] Relationships explicitly stated (one-to-many, many-to-many, etc.)
- [ ] SQL schema or ORM schema code included
- [ ] Every table has: id, created_at, updated_at fields
- [ ] Foreign keys documented
- [ ] ER diagram included in Section 4b
- [ ] Migration strategy defined (Prisma migrate, Drizzle kit, Alembic, etc.)

## Required for Projects WITH Authentication

- [ ] Auth flow described step by step (signup → verify → login → dashboard)
- [ ] Protected vs public routes explicitly listed
- [ ] Roles and permissions defined in a table (if applicable)
- [ ] Session management approach stated (JWT, cookies, etc.)
- [ ] Auth provider webhook handler included in directory structure (if using Clerk/external)
- [ ] Auth flow diagram included in Section 4b

## Security Baseline (required for ALL projects with auth + database)

- [ ] **Row Level Security:** Build order includes RLS implementation immediately after auth setup
- [ ] **RLS mechanism documented:** How user_id filtering works in this stack (global scope, helper, policy, RLS)
- [ ] **RLS exemptions documented:** Tables that are intentionally NOT filtered by user_id (and why)
- [ ] **CORS configured:** Allowed origins whitelist defined in environment setup
- [ ] **Security headers:** All 7 headers listed in deployment or environment setup section
- [ ] **Non-Negotiable Rules include RLS, CORS, and headers** as the first 3 rules in CLAUDE.md

## Required for Projects WITH Payments

- [ ] Pricing model defined (freemium, flat, per-seat, usage-based)
- [ ] Stripe webhook events listed (checkout.session.completed, subscription.updated, etc.)
- [ ] Webhook handler included in directory structure
- [ ] Database fields for billing documented (stripe_customer_id, subscription_status, plan)
- [ ] Feature gating strategy described (how free vs paid features are differentiated)

## Required for Projects WITH Real-Time Features

- [ ] Real-time technology specified and justified
- [ ] Connection/disconnection handling described
- [ ] What data is real-time vs what is fetched normally — clearly separated

## Required for CLI Tools

- [ ] Entry point with shebang (#!/usr/bin/env node) documented
- [ ] Command structure documented (all commands and their flags)
- [ ] Distribution method defined (npm, Homebrew, binary)
- [ ] Interactive vs non-interactive mode handling described

---

## How to Use This Checklist

1. After composing all 16 sections, go through the applicable checklists above
2. If any required item is missing, add it before saving the blueprint
3. Do not ask the user to verify the checklist — this is your quality gate, not theirs
4. If a section is not applicable (e.g., no database), mark the section as "Not applicable" in the blueprint rather than leaving it empty
