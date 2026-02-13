# instrument.md Output Template

Write to `.telemetry/instrument.md` following this structure:

```markdown
# Instrumentation Guide

## Target: [SDK Name]

Generated from tracking-plan.yaml v[version] on [date].

## SDK Setup

### Dependencies
[Package names, install commands]

### Initialization
[Setup code, configuration]

### Environment Variables
| Variable | Purpose | Required |
[Table of env vars needed]

## Identity

### identify()

**Syntax:**
[SDK-specific call signature]

**User Traits:**
| Trait | Type | PII | Notes |
[Mapped from tracking plan entities]

**When to Call:**
[Trigger points]

**Template Code:**
[One real, working example with the plan's actual traits]

### group()

**Syntax:**
[SDK-specific call signature]

**Group Hierarchy:**
| Level | SDK Mapping | ID Source | Parent |
[How plan groups map to SDK]

**Group Traits:**
| Level | Trait | Type | Notes |
[Mapped from tracking plan groups]

**When to Call:**
[Trigger points]

**Template Code:**
[One real, working example per group level]

## Events

### track()

**Syntax:**
[SDK-specific call signature]

**SDK Constraints:**
[e.g., Accoil: no properties — encode in event names]

**Template Code:**
[1-2 representative examples showing the call pattern]

### Group-Level Attribution

[How to attribute events to different group levels in this SDK]
[Template code showing attribution at different levels]

## Architecture

### Client vs Server
[Which calls go where]

### Queues and Batching
[SDK-specific batching/queue behavior]

### Shutdown / Flush
[How to handle graceful shutdown]

### Error Handling
[What to catch, retry behavior]

## SDK-Specific Constraints
[Bullet list of gotchas, limitations, and things to watch for]

## Coverage Gaps
[Note any environments or SDK variants not covered by the reference files.
If the user's stack requires patterns not documented here, flag it.]

---

## Current Implementation
[Preserved from audit, if it existed. Describes how analytics is
currently wired — initialization, routing, call patterns observed.]
```
