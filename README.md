<p align="center">
  <h1 align="center">IAAF</h1>
  <p align="center">
    <strong>Intelligence Artificial Architect Foreman — A Claude Code meta-agent that designs complete software blueprints.</strong>
  </p>
  <p align="center">
    Describe what you want to build. Get a complete blueprint. Let Claude Code build it for you.
  </p>
  <p align="center">
    <a href="README.es.md">Español</a>
  </p>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Claude_Code-Agent-blueviolet?style=for-the-badge" alt="Claude Code Agent" />
  <img src="https://img.shields.io/badge/Blueprints-Markdown-blue?style=for-the-badge" alt="Markdown Blueprints" />
  <img src="https://img.shields.io/badge/License-MIT-green?style=for-the-badge" alt="MIT License" />
  <img src="https://img.shields.io/badge/by-SinapSYS-black?style=for-the-badge" alt="by SinapSYS Ecosistemas" />
</p>

---

# English

## What is IAAF?

Imagine you want to build a house. Before anyone picks up a hammer, you need a **blueprint** — a detailed plan that shows every room, every wall, every pipe, and every wire. Without it, the builders wouldn't know what to do.

**IAAF does this for software.**

You open this project in [Claude Code](https://claude.com/claude-code), tell it what you want to build (a website, an app, a SaaS, an API — anything), and IAAF:

1. **Asks you smart questions** about your idea
2. **Designs the entire system** — database, API, frontend, auth, payments, everything
3. **Generates a blueprint file** (`.md`) so detailed that another Claude Code instance can build the whole thing — step by step, without asking you anything

Think of it like this:

```
You: "I want to build a project management app like Trello"
         ↓
IAAF: asks questions, designs everything
         ↓
The Foreman: injects efficiency rules into the blueprint
         ↓
Blueprint.md: complete plan with tech stack, database schema,
              API routes, component hierarchy, build order...
         ↓
Claude Code: reads the blueprint → builds the entire app (significantly fewer tokens)
```

**You describe the idea. IAAF creates the plan. The Foreman optimizes execution. Claude Code builds it.**

### The Foreman

Every blueprint includes **The Foreman** — a set of behavioral rules embedded in the generated CLAUDE.md that make Claude Code work more efficiently during the build phase:

- **No wasted output** — eliminates sycophantic openers, closing fluff, and unsolicited suggestions
- **Read before write** — reads all relevant files before touching code, reducing unnecessary back-and-forth
- **Test and move on** — when tests pass, moves to the next step without refactoring passing code
- **Smart consolidation** — groups simple operations into single passes, keeps complex tasks incremental

Result: **significant reduction in API costs** during the build phase, without sacrificing code quality. The efficiency rules adapt to each project type — a marketing site gets different optimization rules than a SaaS backend. Actual savings depend on project complexity and conversation length.

---

## How It Works

IAAF follows a 4-phase workflow:

### Phase 1: Discovery
> "What are you building? Who is it for? How big should it be?"

IAAF asks 2-3 questions to understand your idea. Based on your answers, it classifies your project into one of **6 archetypes** (project types).

### Phase 2: Deep Dive
> "Do you need user accounts? Payments? Real-time features?"

Now it asks questions specific to YOUR type of project. It also researches best practices using skills like `/deep-research`.

### Phase 3: Architecture
> "Here's what I'd build: Next.js + Supabase + Clerk + Stripe on Vercel..."

IAAF presents the complete tech stack and architecture with reasons for every decision. You confirm or adjust. For frontend projects, it designs a full visual system (colors, fonts, spacing) using `/ui-ux-pro-max`.

### Phase 4: Generate
IAAF produces the final blueprint — a single `.md` file with **16 sections** covering everything Claude Code needs to build your project from zero to deployed.

---

## The Blueprint: 16 Sections

Every blueprint IAAF generates includes:

| # | Section | What It Contains |
|---|---------|-----------------|
| 1 | Project Overview | Vision, goals, success metrics |
| 2 | Tech Stack | Every technology with rationale |
| 3 | Directory Structure | Full file tree with explanations |
| 4 | Data Model | Entities, fields, relationships, SQL schema |
| 5 | API Design | Routes, endpoints, request/response shapes |
| 6 | Frontend Architecture | Pages, components, state management |
| 7 | Design System | Colors, fonts, spacing, component style |
| 8 | Auth & Authorization | Login flow, roles, permissions |
| 9 | **Build Order** | Step-by-step: what to build first, second, third... |
| 10 | Environment Setup | Prerequisites, env vars, initial commands |
| 11 | Dependencies | Every package with its purpose |
| 12 | Deployment | Hosting, CI/CD, domains |
| 13 | Testing | What to test, which tools, when |
| 14 | Skills to Use | Which Claude Code skills help during the build |
| 15 | **CLAUDE.md** | Complete instructions for the builder — ready to paste |
| 16 | Rules | Non-negotiable constraints for the build |

**Section 9 (Build Order) is the most important.** It tells Claude Code exactly what to build and in what order — so it can work autonomously without asking you questions.

---

## Supported Project Types

IAAF knows how to design **6 types of projects**, each with its own default stack, directory structure, and build order:

| Type | Examples | Default Stack |
|------|----------|--------------|
| **SaaS / Web App** | Trello, Notion, Slack | Next.js + Supabase + Clerk + Stripe + Vercel |
| **Marketing Site** | Landing pages, portfolios | Astro + Tailwind + Sanity + Vercel |
| **Mobile App** | iOS/Android apps | Expo (React Native) + Supabase + NativeWind |
| **API / Backend** | REST APIs, microservices | Hono + Drizzle + PostgreSQL + Railway |
| **Internal Tool** | Admin panels, dashboards | Next.js + shadcn/ui + Prisma + Recharts |
| **Content Platform** | Blogs, docs, CMS | Next.js + Sanity + Algolia + Vercel |

Don't see your project type? IAAF adapts — these are starting points, not limits.

---

## Quick Start

### Prerequisites

- [Claude Code](https://claude.com/claude-code) installed
- A Claude API subscription

### Installation

```bash
# From your project directory:
git clone https://github.com/rickpadro/iaaf.git .iaaf
```

### 4 Modes of Operation

| Mode | Command | For |
|------|---------|-----|
| **GREENFIELD** | `copy .iaaf\CLAUDE.init.md CLAUDE.md` | Build a new project from scratch |
| **IMPROVE** | `copy .iaaf\CLAUDE.improve.md CLAUDE.md` | Fix issues, reduce tech debt, optimize existing project |
| **EXTEND** | `copy .iaaf\CLAUDE.extend.md CLAUDE.md` | Add a new feature to an existing project |
| **FIX** | `copy .iaaf\CLAUDE.fix.md CLAUDE.md` | Diagnose and fix a specific bug |

Then open Claude Code:

```bash
claude
```

### Example: New Project

```bash
mkdir ~/my-new-saas
cd ~/my-new-saas
git clone https://github.com/rickpadro/iaaf.git .iaaf
copy .iaaf\CLAUDE.init.md CLAUDE.md
claude
# Tell IAAF what you want to build. It generates the blueprint.
# Close and reopen Claude Code to start building.
```

### Example: Improve Existing Project

```bash
cd ~/my-existing-project
git clone https://github.com/rickpadro/iaaf.git .iaaf
rename CLAUDE.md CLAUDE.project.md
copy .iaaf\CLAUDE.improve.md CLAUDE.md
claude
# IAAF scans your codebase, diagnoses issues, generates improvement plan.
# When done: rename CLAUDE.project.md CLAUDE.md
```

### Example: Add Feature

```bash
cd ~/my-existing-project
rename CLAUDE.md CLAUDE.project.md
copy .iaaf\CLAUDE.extend.md CLAUDE.md
claude
# Describe the feature. IAAF designs it to fit your existing architecture.
# When done: rename CLAUDE.project.md CLAUDE.md
```

### Example: Fix a Bug

```bash
cd ~/my-existing-project
rename CLAUDE.md CLAUDE.project.md
copy .iaaf\CLAUDE.fix.md CLAUDE.md
claude
# Describe the bug. IAAF diagnoses and fixes it surgically.
# When done: rename CLAUDE.project.md CLAUDE.md
```

> **Note:** For existing projects, always `rename CLAUDE.md CLAUDE.project.md` first. IAAF reads it automatically and respects your project's existing conventions. When the IAAF session ends, restore it with `rename CLAUDE.project.md CLAUDE.md`.

---

## Project Structure

```
iaaf/
├── CLAUDE.md                          # The brain — makes Claude become IAAF
├── knowledge/
│   ├── archetypes/                    # 6 project type templates
│   │   ├── saas-webapp.md             #   SaaS apps
│   │   ├── marketing-site.md          #   Landing pages
│   │   ├── mobile-app.md              #   Mobile apps
│   │   ├── api-backend.md             #   APIs & backends
│   │   ├── internal-tool.md           #   Admin panels
│   │   └── content-platform.md        #   Blogs & CMS
│   ├── building-blocks/               # 12 cross-cutting decision guides
│   │   ├── auth-patterns.md           #   Authentication options
│   │   ├── database-patterns.md       #   Database & ORM choices
│   │   ├── api-design-patterns.md     #   REST vs tRPC vs GraphQL
│   │   ├── frontend-patterns.md       #   Frameworks, styling, state management
│   │   ├── testing-patterns.md        #   Testing strategies
│   │   ├── payments-patterns.md       #   Stripe, billing, subscriptions
│   │   ├── integrations-patterns.md   #   Email, notifications, file storage
│   │   ├── security-ops-patterns.md   #   Security, logging, monitoring
│   │   ├── deployment-patterns.md     #   Hosting, CI/CD, environments
│   │   ├── i18n-patterns.md           #   Internationalization
│   │   ├── performance-patterns.md    #   Web Vitals, optimization
│   │   └── claude-behavior-patterns.md #  The Foreman efficiency rules
│   ├── skills-registry.md             # Maps Claude Code skills to blueprint sections
│   └── stack-compatibility.md         # What works together (and what doesn't)
├── templates/
│   ├── blueprint-template.md          # The 16-section output skeleton
│   └── claude-md-template.md          # Template for the target project's CLAUDE.md
├── questions/
│   ├── phase-1-discovery.md           # Universal discovery questions
│   ├── phase-2-branches.md            # Archetype-specific deep dives
│   └── phase-3-confirmation.md        # Architecture confirmation
└── output/                            # Generated blueprints go here
```

---

## Skills Integration

IAAF leverages Claude Code skills during the design process:

| Skill | Used For |
|-------|----------|
| `/deep-research` | Researching technologies and best practices |
| `/ui-ux-pro-max` | Designing visual systems (colors, fonts, spacing) |
| `/find-skills` | Discovering skills to recommend for the build phase |
| `/playwright-cli` | Analyzing reference sites |

And recommends skills for the **build phase** in the blueprint:

| Skill | Recommended For |
|-------|----------------|
| `/frontend-design` | Building production-grade UIs |
| `/shadcn-ui` | Setting up component libraries |
| `/seo-audit` | SEO auditing after build |
| `/humanizer` | Making content sound natural |

---

## Contributing

Contributions are welcome! You can help by:

- **Adding new archetypes** — new project types in `knowledge/archetypes/`
- **Improving building blocks** — better decision matrices, newer tech options
- **Enhancing the blueprint template** — more sections, better structure
- **Translating** — help make IAAF accessible in more languages

---

## License

MIT License. See [LICENSE](LICENSE) for details.

---

## Built by SinapSYS Ecosistemas
