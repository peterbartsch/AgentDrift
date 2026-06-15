---
name: audit-design-token-drift
description: Guides agents through diffing design tokens across the four canonical surfaces (DESIGN.md, tokens.json, the running CSS, and a live design-system page) and producing a dated drift report. Use when a token-related change has shipped, when a tokens.json metadata.cssSyncRequired queue is non-empty, or on a scheduled cadence (weekly).
---

## Overview

This skill catches the one kind of design-system bug that compounds silently: token drift. A hex value diverges between the human-readable spec (DESIGN.md), the machine-readable bridge (tokens.json), the running code (main.css), and the live demo (design-system.html) — and the longer it lives, the more components reference the wrong value. This skill produces a single dated audit file listing every drift, with file:line citations and a recommended fix per finding.

It is the canonical "Observe" step of a design-system loop when run as a scheduled agent routine. It is also the right thing to run before shipping any change to the running CSS.

## When to Use

Run this skill when:
- A change to any color, spacing, typography, or shadow token is about to ship
- A `tokens.json` `$metadata.cssSyncRequired` queue has any entries
- A new sub-brand / product accent color is introduced
- Weekly, on a schedule (cron / `make audit-tokens`), as drift-prevention infrastructure
- Before publishing a design-system blog post or referenced case study — you cannot claim "0 drift in production" without re-running this

Do NOT run this skill for:
- Component-level visual review (different skill)
- CSS minification verification (a build-gate check, not a token audit)
- WCAG contrast audits (use a dedicated a11y skill)

## Core Process

### 1. Establish the canonical surfaces

The skill diffs four files. If a fifth canonical surface is added (e.g., a Figma Variables export from Tokens Studio), update this skill before adding it to the audit.

| Surface | Typical path | Role |
|---|---|---|
| Spec | `/DESIGN.md` | Human-readable spec; the rules layer |
| Tokens | `/lib/styles/tokens.json` | Machine-readable spec, W3C format; the bridge |
| Code | `/lib/styles/main.css` | Running production code |
| Demo | `/design-system.html` | Live demo page |

### 2. Extract token values from each surface

- **Spec (DESIGN.md)**: grep for hex codes (`#[0-9A-Fa-f]{3,6}`) inside table rows and prose. Tag each finding with its section number so cross-section drift inside the same document is caught.
- **Tokens (tokens.json)**: parse JSON. For each `$value`, resolve `{primitive.x.y.z}` references to a final hex. Build a flat `name → final hex` map.
- **Code (main.css)**: extract the `:root { ... }` block. Capture every `--token-name: value;` declaration. Ignore `@media`-scoped overrides (intentional surface variations, not drift).
- **Demo (design-system.html)**: extract the `:root { ... }` block from its inline `<style>`. Same shape as the code extraction.

### 3. Run four diffs

Compare in this order. Fix in this order too — earlier diffs are higher leverage.

1. **Spec ↔ Spec** (intra-document): does each section agree with the others and with prose mentions? Drift here is the worst — the rules layer cannot disagree with itself.
2. **Spec ↔ Tokens**: every named token in the spec must exist in tokens.json with the same value. Sub-brand accent colors are the most common drift here.
3. **Tokens ↔ Code**: every primitive→semantic chain in tokens.json must resolve to the same hex declared in the CSS. The `cssSyncRequired` queue should be empty after this passes.
4. **Code ↔ Demo**: the live demo page must declare a superset of the code's tokens. Missing tokens here mean components rendered on the demo may silently fall back to defaults.

### 4. Write the audit report

Output one markdown file at `_agents/DESIGN_TOKEN_AUDIT_YYYY-MM.md` (or your project's equivalent) with this structure:

```
# Design Token Audit — YYYY-MM

## Summary
- Total drift findings: N
- Critical (rules layer disagreement): N
- High (canonical-vs-code mismatch): N
- Medium (surface-vs-surface gap): N
- Low (orphaned / undocumented tokens): N

## Findings
For each finding:
- Severity
- Location: file:line ↔ file:line
- Token name
- Values observed (with surface labels)
- Recommended fix (one of: align value, add missing token, remove orphan)

## Statistics
- Token counts per surface
- Coverage percentages

## Auditor / Date
```

### 5. Commit and link

- Commit the audit file
- Link it from any PR that touches tokens or the CSS
- If any Critical findings exist: the change must not ship until they're resolved or explicitly accepted in writing

## Rationalizations

| Rationalization | Reality |
|---|---|
| "It's just one hex value, who cares" | One drift in the rules layer becomes ten in components within a quarter, because every new component reads the wrong value and copies it. |
| "tokens.json is the source of truth — spec drift doesn't matter" | The spec is what humans (and agents) read first when onboarding. If it's wrong, everyone learns the wrong rule. The rules layer is upstream of code. |
| "I'll just fix it inline in the CSS" | Then tokens.json and the spec silently fall behind, and the next agent reads the wrong canonical value. Fix at the upstream surface, then propagate. |
| "The demo page is just a demo, missing tokens don't break production" | They break the demo. Contributors look at the demo to learn what's available. Missing tokens here teach the wrong vocabulary. |
| "We can re-run this later" | The longer drift lives, the more references compound. Catch it within the same change-set or it gets harder to fix every week. |

## Red Flags

- An audit report is committed but no fixes follow within the same PR.
- The audit finds zero drift but the `cssSyncRequired` queue has open entries.
- The spec has hex values *not present* in tokens.json (orphan tokens) without an explicit annotation explaining why.
- A token is corrected in the CSS but the corresponding update to tokens.json is "deferred to next sprint."
- An agent runs this skill, finds drift, and recommends "skip — non-critical" without a written rationale.

## Verification

After completing the skill's process, confirm:

- [ ] An audit file exists, dated this calendar month
- [ ] Every finding has both a `file:line` source citation and a one-line recommended fix
- [ ] The `cssSyncRequired` queue reflects the current state (either empty or matches an open Critical/High finding)
- [ ] If Critical findings exist: an associated commit, PR, or issue is referenced in the audit file
- [ ] The previous month's audit (if any) has been compared — net drift count should trend down or be explained

## Notes

This skill follows the [agent-skills](https://github.com/addyosmani/agent-skills) anatomy. It's a Thios-specific instantiation of a general pattern (rule-layer ↔ implementation-layer drift detection) that other vanilla design systems can adapt. See [`examples/DESIGN_TOKEN_AUDIT_2026-05.md`](../../examples/DESIGN_TOKEN_AUDIT_2026-05.md) for a real first-run output.

The skill is intentionally specific to four surfaces. If a project adds a Figma Variables export or an npm package per Figma's ["Bring your design system" workflow](https://developers.figma.com/docs/code/bring-your-design-system-package/), those become canonical surfaces 5 and 6 — update the skill before they go live.
