# Core Skills

**English** ｜ [🇨🇳 中文](README.zh-CN.md)

The five skills of the AI-PLC pipeline. See the
[root README](../README.md#3-installing-on-amazon-quick-desktop) for installation.

---

## Index

| Skill | Role | Output | references/ | Order |
| --- | --- | --- | --- | --- |
| [`DeepResearch`](DeepResearch/) | Multi-agent research orchestrator | `deepresearch-{slug}-{date}.md` | none | 1 |
| [`brainstorm`](brainstorm/) | Research → PRD | `{slug}-prd-{date}.md` | 3 files | 2 |
| [`plan`](plan/) | PRD → DAG plan | `...-plan.md` | 3 files | 3 |
| [`execute`](execute/) | Plan → artifacts | status write-back + artifacts | 3 files | 4 |
| [`AIPLC`](AIPLC/) | Four-step orchestrator | orchestrates the above | none | 5 (last) |

⚠️ AIPLC declares `depends-on: [deepresearch, brainstorm, plan, execute]`, so it
**must be installed last**.

---

## File layout

Each skill ships in two forms, each with its own purpose:

```
core-skills/
├── DeepResearch.zip     ┐
├── brainstorm.zip       │ for installing
├── plan.zip             │ download → unzip → import into Quick desktop
├── execute.zip          │
├── AIPLC.zip            ┘
│
├── DeepResearch/        ┐
│   └── SKILL.md         │
├── brainstorm/          │ for collaborating
│   ├── SKILL.md         │ diffable / reviewable / full-text searchable in GitHub
│   └── references/      │
│       ├── adversarial-review.md
│       ├── prd-template.md
│       └── pressure-test.md
├── plan/
│   ├── SKILL.md
│   └── references/
│       ├── forcing-questions.md
│       ├── plan-template.md
│       └── validator-checklist.md
├── execute/
│   ├── SKILL.md
│   └── references/
│       ├── oracle-review.md
│       ├── plan-parser.md
│       └── skill-generation.md
└── AIPLC/
    └── SKILL.md
```

The two forms are the same skill in two shapes and **must stay in sync**. See
[CONTRIBUTING.md](../CONTRIBUTING.md#modifying-a-core-skill) for the edit workflow.

---

## About references/

`brainstorm`, `plan`, and `execute` each carry 3 reference files. They are
**lazy-loaded** by design — the skills explicitly instruct not to read them at
session start, only at the phase that needs them, because "context budget is
finite"; loading them early pollutes the context the interrogation and reasoning
depend on.

When importing into Quick desktop, attach the files under `references/` in
addition to `SKILL.md`, or the skill will fail at the phase that needs them.

---

## Verifying zip matches sources

```bash
cd core-skills
for s in DeepResearch brainstorm plan execute AIPLC; do
  echo "=== $s ==="
  diff -r <(unzip -p "$s.zip" SKILL.md) "$s/SKILL.md" > /dev/null \
    && echo "SKILL.md OK" || echo "SKILL.md MISMATCH"
done
```

A MISMATCH means someone updated only one side. Treat the extracted sources as
authoritative and repackage.
