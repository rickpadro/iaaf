# Audit Protocol

After generating the blueprint (Phase 4), run this audit to verify fidelity against the source document and internal consistency. Present the results to the user before finalizing.

---

## When to Audit

- **Always** after Phase 4, before declaring done
- Especially critical when a DDS/PRD/spec was provided in Phase 0

---

## Audit Checklist

### 1. Source Document Fidelity (only if DDS/PRD was provided)

Cross every element from the source document against the blueprint:

| Check | How to verify |
|-------|--------------|
| All modules/features listed in source are present in blueprint | List each module from source, find its corresponding section in blueprint |
| Data model fields match | Compare source entity fields vs blueprint entity fields — flag missing, renamed, or type-changed fields |
| Enums/options match | Compare source enums (types, statuses, categories) vs blueprint enums — flag missing values |
| Numbering/indexing consistency | If source uses 1-based indexing (Phase 1-5), verify blueprint matches or document the mapping |
| Tech stack matches stated preferences | If source specifies a stack, verify blueprint uses it |
| Hosting/deployment matches | If source specifies hosting, verify blueprint targets it |
| User roles match | Compare source user types vs blueprint auth model |
| Pending/future features noted | If source marks features as "pending v1.x" or "future", verify they appear in blueprint roadmap or are explicitly excluded with reason |
| Business rules preserved | Any specific business logic in source (scoring formulas, priority calculations, suggested tasks per phase) must appear in blueprint |
| Criteria of success included | Source success criteria should map to blueprint metrics or be verifiable in the build |

### 2. Internal Consistency

| Check | How to verify |
|-------|--------------|
| Tech stack in Section 2 matches directory structure in Section 3 | If stack says Laravel, structure should show `app/Http/Controllers/`, not `src/app/` |
| Data model in Section 4 matches SQL schema | Every entity field in the table must appear in the SQL/migration |
| Routes in Section 5 match controllers in Section 3 | Every route references a controller that exists in the directory structure |
| Pages in Section 6 match routes in Section 5 | Every page has a route, every route has a page |
| Design system values in Section 7 match CLAUDE.md in Section 15 | Hex values, font names, spacing must be identical |
| Build order steps in Section 9 cover all sections | Every feature in Sections 4-8 must have a corresponding build step |
| Environment variables in Section 10 cover all services in Section 2 | If stack includes Anthropic API, there must be an ANTHROPIC_API_KEY variable |
| Dependencies in Section 11 match stack in Section 2 | Every technology in the stack has corresponding packages listed |
| CLAUDE.md in Section 15 is consistent with the full blueprint | Stack, patterns, rules, design system — all must match |

### 3. Completeness

| Check | How to verify |
|-------|--------------|
| No placeholder text remaining | Search for `{`, `TODO`, `TBD`, `placeholder` in the blueprint |
| All 16 sections filled | No empty sections (mark N/A if genuinely not applicable) |
| Diagrams reference actual entities | Mermaid diagrams use entity names from Section 4, not generic names |
| Build order has specific deliverables per step | Each step says WHAT to build and HOW to verify it works |
| Non-negotiable rules are actionable | Each rule can be verified objectively, not vague |

---

## Audit Report Format

Present the audit results as a table:

```markdown
## Audit Results

### Source Document Fidelity
| Element from DDS | In Blueprint | Status | Notes |
|---|---|---|---|
| {module/feature/field} | {where in blueprint} | OK / MISSING / MISMATCH | {details if not OK} |

### Issues Found
| # | Severity | Issue | Recommendation |
|---|----------|-------|----------------|
| 1 | {High/Medium/Low} | {what's wrong} | {how to fix} |

### Summary
- Elements checked: {N}
- OK: {N}
- Issues: {N} ({high} high, {medium} medium, {low} low)
- Verdict: {PASS / PASS WITH NOTES / NEEDS REVISION}
```

---

## After the Audit

- **PASS**: Deliver the blueprint as-is. Tell the user it's verified.
- **PASS WITH NOTES**: Deliver the blueprint with the notes listed. Let the user decide if they want fixes.
- **NEEDS REVISION**: Fix the issues in the blueprint BEFORE delivering. Re-run the audit on the fixed version.

---

## Severity Definitions

| Severity | Meaning | Action |
|---|---|---|
| **High** | Feature/module from source is missing entirely, or data model is wrong | Must fix before delivering |
| **Medium** | Inconsistency that could cause confusion during build (numbering mismatch, missing env var) | Should fix, or document clearly |
| **Low** | Minor omission, future feature not mentioned, cosmetic issue | Note it, don't block delivery |
