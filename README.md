# Production-Readiness Skills

A library of **Agent Skills** that plug essential reviews and engineering best practices into your AI-assisted development workflow — for Claude Code and Cowork.

When you're vibe coding, the focus is getting the thing *working*. The reviews that make a solution actually production-ready — security, performance, maintainability, and more — get skipped, not because they're hard, but because nobody asked the right questions at the right moment. **These skills ask them for you.**

Drop them into your workflow at any stage and get a consistent, expert-level review pass that catches what slips through when you're moving fast.

---

## The problem this solves

Vibe-coded and quickly-shipped solutions tend to be functional but fragile. The gaps are predictable:

- **Security** holes that never got audited
- **Performance** cliffs nobody load-tested
- **Maintainability** debt that makes the next change painful
- **Flexibility** assumptions that break the moment requirements shift
- **User-friendliness** rough edges that only surface in real use

They're standard checks that get skipped under time pressure or simply because the developer wasn't thinking about them in the moment. This library turns those checks into something you can invoke on demand, so the standard isn't dependent on you being at 100% and all aspects can be confidently accounted for...

---

## What's a skill?

A skill is a folder with a `SKILL.md` file (instructions + YAML frontmatter) that an agent reads to learn a workflow. The agent loads it automatically when your request matches the skill's description, or you invoke it directly. Skills can bundle scripts and reference docs for deeper, deterministic checks.

---

## How it fits your workflow

Use a skill at **any stage** — not just at the end:

- *Mid-build:* "review what I have so far for security gaps"
- *Before merging:* run the relevant review skills as a pre-PR pass
- *On an inherited project:* audit maintainability and flexibility before you touch it

Each skill is a value-add layer on top of whatever you're building — it doesn't get in the way of shipping, it just makes sure shipping doesn't cost you later.

---

## Repo structure

Skills are grouped by **review dimension** — the lens they apply to your work. The grouping below is a starting suggestion, not a fixed schema; expect it to evolve as the library grows. The only firm rule is that each skill lives in its own self-contained folder.

```
skills/                   # repo root
└── <dimension>/          # a review lens (e.g. security, performance) — open and flexible
    └── <skill-name>/
        ├── SKILL.md
        ├── scripts/      # optional: deterministic checks
        ├── references/   # optional: standards/checklists loaded on demand
        └── assets/       # optional: templates
```

Each skill = **one coherent review or practice**, filed under the dimension it serves.

---

Other Useful skills:

| Skill                                                  | Purpose                                         |
|--------------------------------------------------------|-------------------------------------------------|
| [Ponytail](https://github.com/DietrichGebert/ponytail) | Achieve the goal with the fewest possible lines of code without compromising security or core functionality |