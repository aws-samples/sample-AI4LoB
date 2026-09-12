# Extension Skills

**English** ｜ [🇨🇳 中文](README.zh-CN.md)

Home for skills outside the core pipeline: single-purpose tools, customer- or
scenario-specific customizations, and experimental variants of the core skills.

---

## Boundary with core-skills

| Goes in `core-skills/` | Goes in `extend-skills/` |
| --- | --- |
| Part of the four-step AI-PLC pipeline proper | Everything else |
| Changes affect every user; requires merge-request review | Additions are isolated; lighter review |
| Five, and stable | Grows with demand |

**Do not** overwrite a `core-skills/` skill in the name of improvement. Prove the
variant here first; once it works and has real usage evidence, open a merge
request to fold it back into the core.

---

## Adding an extension skill

Mirror the `core-skills/` layout, shipping both the zip and the extracted sources:

```
extend-skills/
├── my-skill.zip
└── my-skill/
    ├── SKILL.md
    └── references/          # optional
        └── whatever.md
```

Naming:

- The directory and zip share one lowercase-hyphenated name.
- One skill per directory, no nesting.
- The instruction file may be named `SKILL.md` or `<skill-name>.md`; both install
  correctly. `core-skills/` uses `SKILL.md`; `protoforge` / `research-to-prd` use
  `<name>.md`. Keep exactly one instruction file per directory.

See [CONTRIBUTING.md](../CONTRIBUTING.md#adding-an-extension-skill) for the full
procedure and what `SKILL.md` should contain.

---

## Index

Add a row here when you contribute a skill.

| Skill | Purpose | Triggers |
| --- | --- | --- |
| [`protoforge`](protoforge/) | Natural-language requirements → development-delivery-grade interactive HTML prototypes. 15-state coverage, three-tier permission model, Web/Mobile dual paradigm, single-file Alpine.js + Tailwind + DaisyUI, zero build | `protoforge`, `生成原型`, `原型生成`, `create prototype`, `interactive mockup`, `交互原型`, `HTML原型` |
| [`research-to-prd`](research-to-prd/) | Two-stage `deepresearch → brainstorm` pipeline, with a depth setting (light 3 agents / standard 6 / deep 8) | `research to prd`, `从研究到PRD`, `研究生成PRD`, `topic to PRD`, `主题转PRD` |

Both were **generated** by the AI4LoB pipeline rather than hand-written — itself a
way of using AI4LoB: build a domain-specific tool, not just the final deliverable.
