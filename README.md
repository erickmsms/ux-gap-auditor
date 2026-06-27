# UX Gap Auditor — a UX-only Claude skill

A Claude skill that walks a **codebase** and maps **user-experience gaps and improvements only** —
usability, interaction cost, flows, information architecture, microcopy, system feedback, and
accessibility-as-operability. It deliberately **ignores the visual/UI layer** (typography, color, spacing,
shadows, radius, dark-mode look, icon styling), because it's built for the case where the aesthetics are
already done and you only want the behavioral/usability furos surfaced.

> **UX only — never UI.** This skill audits how the product *behaves and guides the user*. It will not
> comment on how it *looks*: no typography, color, spacing, shadows, radius, dark-mode appearance, or icon
> styling. If you want a visual restyle, this is the wrong skill — on purpose.

## What it produces
A prioritized, `file:line`-referenced report: findings grouped by Nielsen severity (4 catastrophe → 1
cosmetic), each with the user-impact, the heuristic/lens, and a concrete behavioral fix — plus a "what's
already good" section and an ordered action plan. It does **not** suggest restyling anything.

## Install (Claude Code / any agent that reads SKILL.md)

A skill is just a folder with a `SKILL.md` (plus its `references/`). Put this repo's contents into a
`ux-gap-auditor/` folder inside a `.claude/skills/` directory. You can install it **per project** or **per
user**:

| Scope | Location | Use when |
|---|---|---|
| Project | `<your-repo>/.claude/skills/ux-gap-auditor/` | You want it versioned with one repo / shared with the team |
| Personal (all projects) | `~/.claude/skills/ux-gap-auditor/` | You want it available everywhere you run Claude Code |

### Fastest — clone straight into the skills folder
```bash
# project-level
git clone https://github.com/erickmsms/ux-gap-auditor .claude/skills/ux-gap-auditor

# or personal-level (all your projects)
git clone https://github.com/erickmsms/ux-gap-auditor ~/.claude/skills/ux-gap-auditor
```

### Or download the ZIP
Download this repo as a ZIP, unzip it, and place the folder so the path ends in
`.claude/skills/ux-gap-auditor/SKILL.md`. The final layout must be:

```
.claude/
└── skills/
    └── ux-gap-auditor/
        ├── SKILL.md
        └── references/
            ├── heuristics.md
            ├── states-feedback.md
            ├── microcopy.md
            ├── interaction-laws.md
            ├── flows-ia.md
            └── operable-accessibility.md
```

## Use it
Start (or restart) Claude Code in a project and just ask, in any language — the skill auto-triggers on UX
audit / usability requests:
- "Run a UX audit on this repo" / "Faça uma auditoria de UX neste repositório"
- "Map the UX gaps in the checkout flow" / "Mapeie os furos de UX do fluxo de checkout"
- "Heuristic evaluation of the onboarding" / "Avaliação heurística do onboarding"

It responds in the language you write in. To confirm it loaded, run `/skills` (or ask "which skills do you
have?") and look for **ux-gap-auditor**.

## The six lenses
| Lens | Reference file |
|---|---|
| Usability heuristics (Nielsen, with code detection) | `references/heuristics.md` |
| States & feedback (loading/empty/error/success/disabled) | `references/states-feedback.md` |
| Microcopy & content | `references/microcopy.md` |
| Interaction cost (Hick · Fitts · Miller · Doherty) | `references/interaction-laws.md` |
| Flows & information architecture | `references/flows-ia.md` |
| Operable accessibility (a11y as usability, no contrast/color) | `references/operable-accessibility.md` |

## Scope boundary
In scope: anything that affects how the product *behaves and guides the user*.
Out of scope: anything about how it *looks*. The one nuance — focus *visibility* and contrast: the presence
of a focus indicator is operability (in scope); contrast *ratios* and color are visual (out of scope).

## Credits / provenance
This skill combines and reframes material from two open-source projects, kept to UX-only:
- **claude-design-auditor-skill** by Ashutos1997 (MIT) — source of the code-level detection signals for
  Nielsen heuristics, UI states, and microcopy. The visual categories (typography, color, spacing,
  elevation, etc.) were intentionally dropped.
- **designer-skills** by Owl-Listener (the modular Designer Skills Collection) — source of the heuristic
  evaluation method, the interaction laws (Hick/Fitts/Miller/Doherty), error-handling, loading/feedback
  patterns, information architecture, user flows, onboarding, search UX, and the accessibility test ladder.

All visual/UI guidance from both sources was deliberately excluded to keep this a pure UX audit.
