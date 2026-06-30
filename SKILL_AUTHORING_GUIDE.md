# Skill Authoring Guide

Conventions for authoring skills in this library. Target runtime: **Claude Code + Cowork** (shared `SKILL.md` format).

Every skill here is a **review lens**: it injects a production-readiness check the user would otherwise skip. Two things make a skill belong in this library — it **asks the right questions** for its dimension, and it returns **actionable output** (what's wrong *and* how to fix it). Keep both in mind throughout.

---

## 1. Anatomy

```
skills/<dimension>/<skill-name>/
├── SKILL.md          # required: frontmatter + instructions
├── scripts/          # optional: deterministic checks
├── references/       # optional: standards/checklists loaded on demand
└── assets/           # optional: templates
```

One skill = **one coherent review or practice**. If you're writing "and also" into the description, split it.

---

## 2. Frontmatter

```yaml
---
name: skill-name              # kebab-case, matches folder name
description: <see §3>
version: 0.1.0                # bump on behavior or description changes
---
```

---

## 3. Description (the triggering mechanism)

The description is the **only** thing the agent sees when deciding whether to use the skill. Get it right or the skill never fires.

- **State what it checks AND when to use it.** All "when to use" info lives here, not in the body.
- **Be slightly pushy.** Models under-trigger. Add explicit contexts: *"Use whenever the user mentions security, auth, secrets, or is about to ship/merge — even if they don't ask for a review."*
- **Keep boundaries non-overlapping.** Two skills with overlapping descriptions degrade routing for both. Make each dimension's trigger distinct.
- **Target substantive tasks.** Skills won't fire for trivial one-tool jobs. Aim at real review/audit moments.

---

## 4. Body

- **Imperative voice.** "Check for hardcoded secrets," not "you should check."
- **Explain *why*, don't just legislate.** "Flag X because it leaks under Y" generalizes; a wall of `MUST` doesn't.
- **Lean.** Every line earns its place. Target < 500 lines; push longer standards/checklists into `references/`.
- **Define the output format explicitly** so reviews come back consistent (see §5).

---

## 5. The review itself

This is the heart of a skill in this library.

**Ask the right questions.** Enumerate the failure modes a vibe-coded solution typically misses in this dimension, and have the skill probe each one. The value is surfacing what the user wasn't thinking about — not restating what they already know.

**Be actionable.** Every finding must say what's wrong, why it matters, and how to fix it. A flag without a fix just creates anxiety.

**Suggested finding format** (adapt per dimension):

```markdown
### [severity] <issue>
- **Where:** <file / area>
- **Why it matters:** <concrete risk>
- **Fix:** <specific, actionable step>
```

End reviews with a short summary: what's solid, what's blocking, what's optional polish.

---

## 6. References & scripts

- Push large standards/checklists into `references/`; point to them **explicitly** from the body (*"score auth against `references/auth-checklist.md`"*). Don't rely on auto-loading — the body must be correct even if references aren't pulled in.
- **Deterministic checks → scripts** (e.g. scanning for secrets, dependency audits). Reliable, repeatable, no drift. Keep dependencies minimal and pinned.
- **Judgment calls → prose instructions.**

---

## 7. Generalize, don't overfit

A skill runs against thousands of unseen codebases.

- Resist narrow patches that fix one example and break neighbors.
- When something stubbornly fails, try a different framing or checklist structure, not another `MUST`.
- Keep findings principle-based ("untrusted input reaches a query") rather than tied to one stack's syntax.

---

## 8. Skeleton template

```markdown
---
name: security-review
description: Reviews code for security gaps — auth, secrets, injection, dependency risk. Use whenever the user is about to ship/merge, mentions security/auth/secrets, or asks if their code is safe — even if they don't request a formal review.
version: 0.1.0
---

# Security Review

Surfaces the security gaps that slip through when code is written for "working" first.

## When this applies
<Trigger contexts; point to references if any.>

## What to check
1. <Question / failure mode — and why it matters.>
2. ...

## Output format
For each finding, report: severity, where, why it matters, and the fix.
End with a summary: solid / blocking / optional polish.

## Examples
<Input snippet → expected finding.>
```

---

## 9. Contribution checklist

Matches the checklist in the repo README — every PR is reviewed against it.

- [ ] **One clear review/practice** — folder under the correct `<dimension>/`
- [ ] **Description states what + when** — pushy enough to trigger, distinct from existing skills (no overlap)
- [ ] **Body imperative and lean** (< 500 lines), explains *why*, not just `MUST`
- [ ] **Asks the right questions** — surfaces the gaps a vibe-coded solution typically misses
- [ ] **Actionable output** — flags issues *and* tells the user how to fix them
- [ ] **References pointed to explicitly** — skill works even if they aren't auto-loaded
- [ ] **Deterministic checks live in scripts**; dependencies minimal and pinned
- [ ] **Self-contained** — no dependencies on other skills' files
- [ ] **`version` set/bumped** — no surprising or undocumented behavior changes

## Runtime & installation context

This library targets agents that read the shared `SKILL.md` format. A skill is
installed by placing the skill folder itself in the agent's skills directory.

**Install one skill** by copying the skill folder, not the whole repo:

```bash
# Personal: available across all your projects
cp -r skills/<dimension>/<skill-name> ~/.claude/skills/

# Project-scoped: only in this repo
cp -r skills/<dimension>/<skill-name> .claude/skills/
```

**Or clone the whole library** as personal skills:

```bash
git clone <repo-url> ~/.claude/skills/production-readiness-skills
```

Start a new Claude Code session, then confirm it loaded:

```text
/skills
```

A skill named `security-review` becomes `/security-review`, and also auto-loads
whenever a request matches its description.

Cowork uses the same `SKILL.md` format; install by placing the skill folder in
Cowork's skills directory the same way.

### Using a skill

- **Automatic:** describe the task. If it matches a skill's description, the
  agent loads it (`check this for security issues` activates `security-review`).
- **Direct:** invoke the slash command, such as `/security-review`.

If a skill is not activating, check that its description matches the user's
phrasing and that the session was restarted after installing.

### Skill catalog

The public catalog lives in `README.md`. When adding a skill, add it to the
catalog with its dimension, skill name, and the production-readiness check it
performs.

---

## Related skill registries

This repo is focused on production-readiness review lenses. Related registries
can cover adjacent agent workflows without being folded into this library.
