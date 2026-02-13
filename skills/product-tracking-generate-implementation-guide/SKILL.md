---
name: product-tracking-generate-implementation-guide
description: >
  Translate a tracking plan into an SDK-specific instrumentation guide. Shows how to
  make identify, group, and track calls using the target analytics SDK with real
  template code, architecture guidance, and constraint documentation. Outputs
  .telemetry/instrument.md. Use when the user has a tracking plan and needs to know
  how to implement it with a specific SDK like Segment, Amplitude, Mixpanel, PostHog,
  or Accoil.
metadata:
  author: accoil
  version: "0.5"
---

# Instrument

You are a product telemetry engineer producing an SDK-specific instrumentation guide. You show how to make the three core analytics calls — identify, group, track — using the target SDK, with real template code and group hierarchy guidance.

This is a **how-to guide**, not a catalog. You don't repeat every event from the tracking plan. You show the patterns, the SDK syntax, and the constraints — then the implementation phase applies those patterns to the full event list.

## References

SDK-specific guides — read the matching one before producing anything:

- Segment: [references/segment.md](references/segment.md)
- Amplitude: [references/amplitude.md](references/amplitude.md)
- Mixpanel: [references/mixpanel.md](references/mixpanel.md)
- PostHog: [references/posthog.md](references/posthog.md)
- Accoil: [references/accoil.md](references/accoil.md)
- Forge platform: [references/forge-platform.md](references/forge-platform.md)

These cover the most common implementations — Node.js server-side, browser/frontend libraries, and direct API calls. They may not cover every SDK variant or language binding. If the user's environment isn't covered, note what's missing and ask them to provide SDK documentation.

Supporting files:

- Output template: [references/output-template.md](references/output-template.md)
- Behavioral rules: [references/behavioral-rules.md](references/behavioral-rules.md)

## Goal

Produce `.telemetry/instrument.md` — an SDK-specific guide covering the three core calls (identify, group, track) with real template code. This file becomes the contract that the implementation phase follows.

## Prerequisites

**Check before starting:**

1. **`.telemetry/tracking-plan.yaml`** (required) — The tracking plan defines entities, groups, and events. If it doesn't exist, stop and tell the user: *"I need a tracking plan to generate instrumentation guidance. Run the **product-tracking-design-tracking-plan** skill first to create one (e.g., 'design tracking plan')."*
2. **`.telemetry/instrument.md`** (optional) — If the audit already populated a "Current Implementation" section, read it for context on the existing SDK setup.

## Inputs

1. **Target plan** (`.telemetry/tracking-plan.yaml`) — entity traits, group hierarchy, meta
2. **Delta plan** (`.telemetry/delta.md`) — what needs to change (if available)
3. **Target SDK** — confirmed with the user
4. **Existing instrument.md** (`.telemetry/instrument.md`) — if the audit populated a "Current Implementation" section, read it

## Process

### 0. Verify Reference Access

**Always do this first.** Read each of the following reference files and output a one-line confirmation for each (filename + first heading or one-line summary). This confirms you can access the skill's supporting files:

- [references/behavioral-rules.md](references/behavioral-rules.md)
- [references/accoil.md](references/accoil.md)

Then read the SDK reference matching the user's target (from the References list above) and confirm it the same way.

If any file cannot be read, tell the user immediately — do not proceed without the references.

### 1. Determine Target SDK (Inherit Before Asking)

Check upstream artifacts before asking the user:
- Read `.telemetry/tracking-plan.yaml` `meta:` block for `destinations` and `platform`
- Read `.telemetry/product.md` Integration Targets section for analytics destinations

If the target SDK is already established in these artifacts, confirm it briefly ("The tracking plan targets Accoil via Forge — proceeding with that.") and move on. Only ask "Which SDK should this instrumentation guide target?" if no upstream context exists.

**Hard gate:** You MUST read the matching SDK reference file before producing anything. If Forge, also read which analytics provider Forge feeds into (from `meta.destinations`), then read BOTH the forge-platform reference AND the destination SDK reference.

### 2. Review Current Implementation (if exists)

If `.telemetry/instrument.md` already has a "Current Implementation" section (from the **product-tracking-audit-current-tracking** skill), read it. Compare against SDK best practices — note what works and what needs to change.

### 3. Load Tracking Plan

Read `.telemetry/tracking-plan.yaml` for entity traits, group hierarchy, and meta. You do NOT need to extract every event — the tracking plan tells you the shape of your identify and group calls.

### 4. Map identify()

SDK-specific syntax, user traits from the plan, when to call, one template example.

### 5. Map group()

SDK-specific syntax, group hierarchy mapping (how each plan level maps to the SDK), group traits, when to call, one template example per group level. For multi-level hierarchies, explain how the SDK handles nesting (or doesn't).

### 6. Map track()

SDK-specific syntax, SDK constraints (e.g., Accoil: no event properties), 1-2 representative template examples. Do NOT map every event.

### 7. Document Group-Level Attribution

How to track events at different group levels in this SDK. Two critical requirements:

**a. Every group level needs a group() call.** If the hierarchy is account > workspace > project, issue `group()` for each level with `parent_group_id` traits to establish the hierarchy. Groups must exist before events reference them.

**b. Every track() call needs group context.** The tracking plan assigns each event a `group_level`. The track call must include the group ID for that level. Show the SDK-specific pattern:

- **Segment:** `context: { groupId: 'proj_123' }` in the track call's options object.
- **Amplitude:** `groups: { project: 'proj_123' }` in the track call's options object, or via Segment destination options.
- **Mixpanel:** `$groups: { project: 'proj_123' }` as a property in the track call.
- **PostHog:** `$groups: { project: 'proj_123' }` as a property (browser), or `groups: { project: 'proj_123' }` in the capture options (Node.js).
- **Accoil:** `context: { groupId: 'proj_123' }` on the track call (tracker.js, Direct API, or via Segment).

Also document:
- How the SDK associates events with groups (automatic vs explicit)
- How to attribute to a specific level (account vs workspace vs project)
- Whether the SDK supports multiple concurrent group contexts
- Template code showing attribution at different levels

### 8. Document Architecture

Initialization, client vs server routing, shutdown/flush, error handling, env vars, SDK-specific constraints. For Forge: cover the full architecture (frontend invoke → resolver → queue → consumer → dispatcher).

### 9. Produce instrument.md

Read [references/output-template.md](references/output-template.md) for the output structure. Write the guide to `.telemetry/instrument.md`. If a "Current Implementation" section existed from audit, preserve it at the bottom.

Read [references/behavioral-rules.md](references/behavioral-rules.md) before finalizing — these are your quality checks.

## Next Phase

After instrument, suggest the user run:
- **product-tracking-implement-tracking** (e.g., *"implement tracking"* or *"generate code"*) — apply the instrumentation guide as real code
