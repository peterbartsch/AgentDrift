# AgentDrift

Reusable agent workflows written as portable Markdown, following the [agent-skills](https://github.com/addyosmani/agent-skills) anatomy (MIT). A skill is a workflow an agent *follows* — not a reference doc it reads.

These were extracted from real use building [Thios](https://thios.co), a product run on a five-surface design system where one designer reviews the work and agents do the implementation. The skills reference design-system concepts (a markdown spec, a `tokens.json`, running CSS, a live demo page), but the *shape* of each workflow generalizes to any vanilla-CSS design system.

## Skills

| Skill | What it does |
|---|---|
| [`audit-design-token-drift`](skills/audit-design-token-drift/SKILL.md) | Diffs design tokens across four canonical surfaces (spec ↔ tokens.json ↔ CSS ↔ live demo) and produces a dated drift report with `file:line` citations and a recommended fix per finding. |

See [`examples/DESIGN_TOKEN_AUDIT_2026-05.md`](examples/DESIGN_TOKEN_AUDIT_2026-05.md) for a real first-run output — it caught 7 drift findings, including a spec that contradicted itself on the canonical brand color and a documented color absent from the token file.

## Roadmap

| Skill | Purpose |
|---|---|
| `audit-cad-hygiene` | Naming conventions, mate health, and build-correctness checks on a parametric CAD document. |
| `audit-component-consistency` | Diff documented components in the live demo vs. actual production usage. |
| `sync-tokens-to-figma` | Push `tokens.json` updates to Figma Variables via the Tokens Studio plugin. |
| `verify-css-sync` | Wrap a multi-surface CSS-sync gate as a structured skill with audit output. |

## Skill anatomy

Every `SKILL.md` here follows the [skill-anatomy spec](https://github.com/addyosmani/agent-skills/blob/main/docs/skill-anatomy.md):

1. **Frontmatter** (YAML) — `name`, `description` with trigger conditions
2. **Overview** — what the skill does, in 1–2 sentences
3. **When to Use** — explicit triggers and non-triggers
4. **Core Process** — numbered, actionable steps
5. **Rationalizations** — common excuses to skip steps, and the counter-arguments
6. **Red Flags** — observable signs the skill is being violated
7. **Verification** — exit-criteria checklist with evidence requirements

## Why markdown skills

Markdown-as-instruction-layer is the convergent shape across the field: Anthropic's published Skill format, Figma's [`Guidelines.md`](https://developers.figma.com/docs/code/bring-your-design-system-package/) for design-system packages, and [addyosmani/agent-skills](https://github.com/addyosmani/agent-skills). Plain Markdown works with any agent that accepts system prompts or instruction files.

## License

[MIT](LICENSE) — use, adapt, and PR freely.

---

Maintained by [Peter Bartsch](https://petebartsch.com).
