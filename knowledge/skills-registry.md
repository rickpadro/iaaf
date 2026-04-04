# Skills Registry

Maps available Claude Code skills to blueprint sections. Use this during Phase 4 (Generate) to recommend the right skills for the build phase.

## Skills for Blueprint Recommendations

| Skill | Blueprint Section | When to Recommend | What It Does |
|-------|------------------|-------------------|-------------|
| `/frontend-design` | Frontend Architecture, Build Order | Any project with a UI | Creates distinctive, production-grade frontend interfaces |
| `/shadcn-ui` | Tech Stack, Build Order | When shadcn/ui is chosen as component library | Search and implement shadcn components |
| `/ui-ux-pro-max` | Design System | Any project with a UI | 50+ styles, 161 palettes, 57 font pairings, UX guidelines |
| `/seo-audit` | Deployment Strategy, Build Order | Marketing sites, content platforms, any public-facing site | Full technical SEO audit |
| `/deep-research` | Tech Stack (during design) | When comparing technologies or unfamiliar tools | Systematic multi-angle web research |
| `/find-skills` | Skills to Use (during design) | Always run once to discover build-phase skills | Discovers installable skills |
| `/humanizer` | Build Order (content step) | When project includes marketing copy or content writing | Removes AI-generated writing patterns |
| `/pdf-design` | Build Order | When project generates PDFs (reports, invoices, docs) | Professional PDF creation |
| `/playwright-cli` | Testing Strategy, Build Order | When E2E testing is in scope | Browser automation and testing |
| `/web-reader` | Build Order | When project needs web scraping or content extraction | Web page content extraction |
| `/chrome-bridge-automation` | Build Order | When project automates browser workflows | Vision-driven browser automation |

## Skills The Architect May Use During Design (optional — only if installed)

These skills are NOT required. The Architect works without them. Only use if available in the environment.

| Skill | Phase | Purpose | If not available |
|-------|-------|---------|-----------------|
| `/deep-research` | Phase 2 | Compare technologies, research patterns | Use your own knowledge — it's sufficient for most decisions |
| `/ui-ux-pro-max` | Phase 3 | Design the visual system | Design it yourself: pick colors, fonts, spacing based on project type |
| `/find-skills` | Phase 2 | Discover skills for the build phase | Skip — recommend skills from this registry instead |
| `/playwright-cli` | Phase 3 | Screenshot reference sites | Ask user to describe what they like about the reference site |

## How to Include in Blueprint

In Section 14 (Skills to Use During Build), list each relevant skill with:
1. **Skill name** — the slash command
2. **When to use** — which build step (reference the build order)
3. **Why** — what it helps accomplish

Example:
```markdown
| Skill | When to Use | Why |
|-------|-------------|-----|
| /frontend-design | Steps 4, 5, 10 (layouts, core feature UI, landing page) | Production-grade, distinctive UI |
| /shadcn-ui | Step 1 (scaffolding) | Set up and customize component library |
| /seo-audit | Step 11 (polish) | Audit SEO before launch |
```
