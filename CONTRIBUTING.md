# Contributing

**English** ｜ [🇨🇳 中文](CONTRIBUTING.zh-CN.md)

Contributions of new skills are welcome. Read the relevant section before you
start.

---

## Quick start

```bash
git clone https://github.com/aws-samples/sample-ai-plc-practices.git
cd sample-ai-plc-practices
git checkout -b feature/<short-description>
# ... make changes ...
git add .
git commit -m "core-skills: fix the wave example in plan"
git push -u origin feature/<short-description>
# Open a Pull Request on GitHub
```

---

## Branches and commits

- Never push straight to `main`; use a feature branch and a pull request.
- Branch naming: `feature/...`, `fix/...`, `docs/...`.
- Use `<scope>: <what changed>` for commit messages, where the scope is one of
  `core-skills` / `extend-skills` / `docs` / `readme`.

Explain *why* in the description, not just *what* — `git diff` already shows
*what* changed.

---

## Modifying a core skill

Skills under `core-skills/` affect every user, so changes require Pull Request
review.

### ⚠️ Keep zip and sources in sync

Each skill exists as `<name>.zip` (for installing) and `<name>/` (for
collaborating). They **must** match; updating only one side ships a stale version
to colleagues who download the zip.

The correct order:

```bash
cd core-skills

# 1. Edit the extracted sources first — they are authoritative
vim brainstorm/SKILL.md

# 2. Repackage; -r to include references/
rm brainstorm.zip
cd brainstorm && zip -r ../brainstorm.zip . -x '.*' && cd ..

# 3. Verify the zip contents
unzip -l brainstorm.zip

# 4. Commit both sides together
cd .. && git add core-skills/brainstorm core-skills/brainstorm.zip
git commit -m "core-skills: <what changed, and why>"
```

`-x '.*'` excludes hidden files such as the macOS `.DS_Store`.

### Verify they match

```bash
cd core-skills
for s in DeepResearch brainstorm plan execute AIPLC; do
  diff <(unzip -p "$s.zip" SKILL.md) "$s/SKILL.md" > /dev/null \
    && echo "$s OK" || echo "$s MISMATCH"
done
```

### Before you touch a core skill

Several design choices are **deliberate**, not oversights. Make sure you
understand the rationale before changing them:

- **Stable IDs are never recycled** — gaps in the numbering are correct.
  `R#/A#/F#/AE#/U#` are referenced across sessions and across skills;
  renumbering breaks the traceability chain.
- **Six-value state machine** — `PLANNED` / `IN_PROGRESS` / `DONE` /
  `DONE_WITH_CONCERNS` / `BLOCKED` / `NEEDS_CONTEXT`. Adding a seventh value
  breaks downstream parsers.
- **Lazy-loaded reference files** — not reading `references/` at session start
  protects the context budget.
- **Hard boundaries** — brainstorm does not write code, plan does not execute,
  execute does not author artifacts itself. Relaxing any of these gates makes
  pipeline output unpredictable.

If you must change one of the above, spell out the blast radius in the Merge
Request description and ping the repository owner.

Verify on a real machine afterwards: import into Quick desktop, run it once, and
paste the result into the MR.

---

## Adding an extension skill

New skills go in `extend-skills/`, not directly into `core-skills/`. For the
boundary between the two, see
[`extend-skills/README.md`](extend-skills/README.md).

### Layout

```
extend-skills/
├── my-skill.zip
└── my-skill/
    ├── SKILL.md             # or my-skill.md — either name works
    └── references/          # optional
```

Use lowercase-hyphenated names; the directory and zip share a name. The
instruction file may be named `SKILL.md` or `<skill-name>.md` — both install
correctly (`core-skills/` uses the former, `protoforge` / `research-to-prd` use
the latter). Keep exactly one instruction file per directory.

### What SKILL.md should contain

Follow the existing skills in `core-skills/`. At minimum:

- **name / description** — the description drives Quick's automatic skill
  selection, so state the triggers clearly.
- **role** — what role the skill plays, in a sentence or two.
- **triggers** — list them explicitly, and also list what should *not* trigger
  the skill.
- **inputs / outputs** — pin down output filenames; do not leave them to the
  agent's discretion.
- **phases** — break the work into steps, each with its entry condition and
  output.
- **abort conditions** — when to stop and ask the user rather than guess.
- **anti-patterns** — state explicitly what is forbidden.

### Before submitting

1. Import and run it successfully on Quick desktop.
2. Confirm the zip matches the sources.
3. Register a row in the index table in
   [`extend-skills/README.md`](extend-skills/README.md).
4. In the Pull Request, explain what problem it solves and how you verified it.

---

## ⚠️ Never commit sensitive data

This is a public repository. Never commit: customer names, contract values, AWS
account IDs / ARNs / live endpoints, any credential, personal names, emails, or
phone numbers, NDA materials, or samples of anyone's business data.

Rule of thumb: assume anything here gets screenshotted and shared anywhere.
Would that create a problem?

Deleting sensitive content in a follow-up commit does not remove it from Git
history. Open an issue immediately if something slips through.

---

## Review criteria

Pull Requests are reviewed against the following points:

| Item | Requirement |
| --- | --- |
| Correctness | The modified skill runs successfully on a real Quick desktop |
| Sync | The zip matches the sources |
| Sanitization | No sensitive data anywhere in the change |
| Bilingual | Every user-facing doc ships as a pair — `X.md` (English) and `X.zh-CN.md` (Chinese) — with a language switcher on line 3, and both must be updated together. |
| Index | New content is registered in the corresponding README index table |
| Clarity | The PR description explains *why*, not just *what* |

---

## Contact

Please open an issue or pull request for questions and suggestions.
