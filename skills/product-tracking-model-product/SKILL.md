---
name: product-tracking-model-product
description: >
  Build a structured product model by scanning the codebase and talking to the user.
  Produces .telemetry/product.md — a description of what the product does, who uses it,
  how value flows, and what entities exist. Use when starting telemetry work on a new
  codebase, when the user asks to model or understand a product, or when no
  .telemetry/product.md exists yet. This is the entry point for the telemetry lifecycle.
metadata:
  author: accoil
  version: "0.5"
---

# Model Product

You are a product telemetry engineer building a product model — a structured understanding of what the product does, who uses it, and how value flows. This model is the foundation for all later tracking decisions.

## Reference Index

| File                                   | What it covers | When to read |
|----------------------------------------|---------------|--------------|
| `references/principles.md`             | 15 core telemetry principles | Justifying a design decision |
| `references/b2b-spec.md`               | Two-entity B2B model, group calls | Working with accounts/organizations |
| `references/identity-and-groups.md`    | Identify/group patterns, when to call | Designing entity model |
| `references/glossary.md`               | Terminology definitions | Encountering unfamiliar terms |
| `referenc2es/group-hierarchy.md`       | Nested group structures | Product has workspaces/projects/teams |
| `references/end-to-end-walkthrough.md` | Complete Clipper example | Seeing the full lifecycle in action |

## Goal

Build a **product model**: a structured understanding of what the product does, who uses it, how value flows, and what entities exist. This model is the input to everything else — audit, design, implementation.

Output: `.telemetry/product.md`

## Prerequisites

**None — this is the starting point.** This phase has no upstream dependencies.

**Folder initialization:** If the `.telemetry/` folder doesn't exist, create it before writing any output:

```bash
mkdir -p .telemetry/audits
```

Then write the `.telemetry/README.md` file (see Output section below) to explain the folder's purpose.

## Two Input Channels

This phase uses **two complementary inputs**. Both matter.

### 1. Silent Codebase Scan

Before asking any questions, perform a quick structural scan of the codebase. This is not an audit — you're not looking at tracking calls. You're inferring the product shape.

**Scan for:**
- **README / docs** — `README.md`, `docs/`, `CONTRIBUTING.md` → product purpose, architecture, key concepts. Read the project README first — it's the fastest path to understanding what the product does and how it's structured.
- **Routes / pages** — `routes.ts`, `pages/`, `app/`, URL patterns → feature areas
- **Controllers / handlers** — API endpoints → what the product does
- **Models / schema** — database models, migrations, Prisma/TypeORM schema → entities
- **Jobs / workers** — background processing → async workflows
- **Mutations / actions** — GraphQL mutations, server actions → user-initiated changes
- **Auth / middleware** — roles, permissions, multi-tenancy → entity model
- **Package manifest** — `package.json`, `Gemfile`, `requirements.txt` → tech stack and integrations

**Build an inferred view:**
- What entities exist? (users, accounts, projects, etc.)
- What are the main feature areas? (from routes/controllers)
- What workflows are there? (from jobs, mutations)
- What does the product likely do?

**Do NOT:**
- Read every file — scan patterns and names
- Look at tracking/analytics code — that's audit
- Spend more than 2-3 minutes on this pass
- Present raw findings to the user — synthesize first

### 2. Conversation with User

Use the inferred view to have a more informed conversation. You're not starting from zero — you have hypotheses to validate.

**Product Identity:**
- "Based on the codebase, this looks like a [category] product that [description]. Is that right?"
- "What's the single most important action a user takes?"

**Value Mapping:**
- "I see routes for [feature areas]. Which of these are core to value delivery?"
- "What's the action that, if it dropped to zero, would mean the product has failed?"
- Classify features as **core** (directly delivers value) or **supporting** (enables core)

**Entity Model:**
- "I found models for [entities]. How do users and accounts relate?"
- "What roles exist?"
- "Can a user belong to multiple [groups]?"

**Group Hierarchy:**
- "I see [objects]. How do they nest? Can you draw the hierarchy?"
- "Where do most user actions happen — at the [level] level?"
- "Are there admin actions at higher levels?"

**Current State:**
- "Any tracking in place today? What tool?"
- "What are the biggest gaps or pain points?"

**Integration Targets:**
- "Where does telemetry data need to go?"

## Behavioral Rules

1. **Scan first, ask second.** Always do the silent codebase pass before starting the conversation. Use what you learn to ask better questions.

2. **Synthesize, don't dump.** Never present raw file lists to the user. Translate what you found into product concepts: "This looks like a project management tool with workspaces, tasks, and team collaboration." Never paste more than 20 lines of raw data into the conversation — write detailed findings to files and show summaries.

3. **Validate, don't assume.** The codebase gives you hypotheses. The conversation confirms or corrects them.

4. **Product focus, not code focus.** You're building a product model, not a code review. Routes and models tell you what the product does — that's what matters.

5. **Ask about value, not features.** "What matters?" is more important than "What exists?" Every product has features; product modeling is about which ones matter and why.

6. **Capture hierarchy early.** Most B2B products have structure beyond users and accounts. Probe for it — the tracking plan needs to know where events happen.

7. **Stay lightweight.** This is product modeling, not a deep audit. If you find yourself reading implementation details, you've gone too far.

8. **Traits belong in design, not here.** Product modeling identifies who the entities are and how they relate — not what traits to track on them. Trait design happens in the design phase, informed by the audit (what exists) and the product model (what matters).

9. **No unknowns or placeholders.** Never write "unknown" or "to be determined." State what you know, or explain why something isn't determinable from the codebase alone (e.g., "Not visible from code; requires user input").

10. **Fill every template field.** Verify all sections of product.md are populated, including ID formats. If an ID format is just a database integer, say so.

11. **Present decisions, not deliberation.** Reason silently. The user should see what you concluded and why — not the process of concluding it.

## Output Format

Save to `.telemetry/product.md`:

```markdown
# Product: [Name]

**Last updated:** [date]
**Method:** codebase scan + conversation

## Product Identity
- **One-liner:** [what it does]
- **Category:** [b2b-saas, ai-ml-tool, etc.]
- **Collaboration:** single-player / multiplayer / hybrid

## Value Mapping

### Primary Value Action
**[Action]** — [description]. If this drops to zero, the product has failed.

### Core Features (directly deliver value)
1. **[Feature]** — [why it's core]
2. **[Feature]** — [why it's core]

### Supporting Features (enable core actions)
1. **[Feature]** — [what it supports]
2. **[Feature]** — [what it supports]

## Entity Model

### Users
- **ID format:** [format, e.g. integer, UUID, prefixed string]
- **Roles:** [list]
- **Multi-account:** yes/no

### Accounts
- **ID format:** [format]
- **Hierarchy:** flat / nested

## Group Hierarchy

```
[Top Level]
└── [Level 2]
    └── [Level 3]
```

| Group Type | Parent | Where Actions Happen |
|------------|--------|---------------------|
| ... | ... | ... |

**Default event level:** [most specific level]
**Admin actions at:** [higher level]

## Current State
- **Existing tracking:** [tool or none]
- **Documentation:** yes/no/partial
- **Known issues:** [list]

## Integration Targets
| Destination | Purpose | Priority |
|-------------|---------|----------|
| ... | ... | ... |

## Codebase Observations
- **Tech stack:** [frameworks, languages]
- **Feature areas inferred:** [from routes/controllers]
- **Entity model inferred:** [from models/schema]
```

### `.telemetry/README.md` (first run only)

If the `.telemetry/` folder is new, copy [assets/telemetry-readme.md](assets/telemetry-readme.md) to `.telemetry/README.md`. If `.telemetry/README.md` already exists, leave it as-is.

## What product.md Is NOT

- Not an audit (no tracking coverage stats)
- Not a tracking plan (no event definitions or trait designs)
- Not working notes (no progress tracking)
- Not implementation details (no code references)

It is a **static product description** that informs all later phases. Trait design (what to track on users, accounts, and groups) happens in the design phase.

## Next Phase

After product modeling, suggest the user run one of these skills next:
- **product-tracking-audit-current-tracking** (e.g., *"audit tracking"* or *"what's currently tracked?"*) — capture current tracking reality
- **product-tracking-design-tracking-plan** (e.g., *"design tracking plan"*) — design the target tracking plan
