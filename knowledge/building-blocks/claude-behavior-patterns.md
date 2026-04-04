# Building Block: Claude Code Behavior Optimization (The Foreman)

Behavioral rules that optimize how Claude Code works during the build phase. These reduce output token waste and unnecessary API turns without sacrificing code quality.

## Universal Rules (Include in ALL Blueprints)

These 9 rules apply regardless of project type. They go in the `## The Foreman` section of the target project's CLAUDE.md.

1. **Be concise in output but thorough in reasoning.** No sycophantic openers ("Sure!", "Great question!") or closing fluff ("Hope this helps!").
2. **Think before acting.** Read existing files before writing code. Understand what exists before changing anything.
3. **Prefer editing over rewriting.** Use the Edit tool when possible instead of rewriting entire files.
4. **Do not re-read files** already in context unless the file may have changed.
5. **Test before declaring done.** If tests pass, move to the next step immediately.
6. **Do not refactor passing code.** No polishing, improving, or reorganizing code that already works.
7. **No unsolicited suggestions** beyond the scope of the current build step.
8. **Keep solutions simple and direct.** Three similar lines > premature abstraction.
9. **User instructions always override this file.**

## Per-Archetype Additions

Add these rules alongside the universal rules based on the project archetype.

### SaaS Web App / Internal Tool

```markdown
- Return code first. Explanation only if non-obvious.
- State the bug. Show the fix. Stop.
- No compliments on the code.
- When building CRUD operations, write the route handler + validation + service logic together in one pass.
- Do not add error handling for scenarios that cannot happen given the current data model.
```

### API / Backend Service

```markdown
- All API responses must follow the defined response shape. No variations.
- Write integration tests alongside route handlers, not after all routes are built.
- Validate all inputs at the API boundary with the chosen validation library. No validation deep in business logic.
- Do not add middleware you were not asked to add (logging, metrics, tracing) unless specified in the build order.
```

### Marketing Site / Landing Page

```markdown
- Optimize for Core Web Vitals during every component build. No unnecessary JavaScript.
- Do not add JavaScript to a component unless interactivity is explicitly required.
- Build each section (Hero, Features, CTA) as a complete, self-contained unit before moving to the next.
- Inline critical CSS. Defer non-critical assets.
```

### Mobile App

```markdown
- Test on both iOS and Android considerations before marking a step complete.
- Check platform-specific behavior: safe areas, keyboard handling, gesture conflicts.
- Do not install packages without first checking React Native compatibility.
- Write hooks for data fetching (TanStack Query) alongside the screen that uses them, not separately.
```

### Content Platform / CMS

```markdown
- Server-render all content. No client-side markdown rendering.
- Build content display pages before content management pages.
- SEO metadata must be implemented on every content page at creation time, not as a separate polish step.
- Use ISR (Incremental Static Regeneration) by default for content pages.
```

## Build Efficiency Guidelines

These go in the blueprint's Build Order section (Section 9) as a preamble before the numbered steps.

```markdown
### Build Efficiency Guidelines
- READ all relevant files at the start of each step before writing anything.
- Do not re-read files already in context.
- When tests pass, move to the next step. Do not refactor passing code.
- Do not narrate what you are about to do. Just do it.
- For simple sub-tasks (types, validation schemas, config files): write them together in one pass.
- For complex sub-tasks (auth flow, payment integration, real-time): write and test incrementally.
```

## Tool Call Awareness

These go in the target project's CLAUDE.md under `## Tool Call Awareness`.

```markdown
- Prefer fewer, larger operations over many small ones.
- Read all relevant files at the start of each build step, not one at a time.
- Write complete files in one operation when possible.
- If a step is taking more than 15 tool calls, reassess your approach.
- This is a guideline, not a hard limit. Correctness over speed.
```

## When to Load This Building Block

Always. The Architect should read this file during Phase 4 (Generate) and incorporate the appropriate rules into:
- Section 9 (Build Order) — the Build Efficiency Guidelines
- Section 15 (CLAUDE.md) — the universal rules + archetype-specific rules + tool call awareness
