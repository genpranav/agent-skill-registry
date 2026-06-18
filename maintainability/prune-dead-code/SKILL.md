---
name: prune-dead-code
description: >-
  Review a codebase for dead code (unreachable functions, unused
  imports/variables, orphaned files, leftover implementations after a refactor,
  unused dependencies) in any language, produce a tiered removal plan, and carry
  out the removal once the user approves. Use this skill whenever the user asks
  to find or remove dead code, unused code, leftover or orphaned implementations,
  clean up after reworking or replacing a feature, prune unused
  imports/exports/files, or trim unused dependencies, even if the exact phrase
  "dead code" is not used. Removal is destructive, so this skill reports its plan
  first and deletes only after explicit approval; it is invoked manually by the
  user, never automatically.
version: 0.1.0
---

# Dead Code Review & Removal

This skill is a maintainability review lens: it audits a codebase for
dead code, reports a **tiered removal plan** the user can act on, and then —
only after approval — carries out the removal safely. The hard, dangerous part
is not deleting, it is deciding what is *actually* dead. Detection is
stack-specific and false positives break the build (or worse, break things
silently). So this skill's job is to **dispatch to the right detector, apply
conservative tiered judgment, present that judgment as a reviewable plan, and
remove only what the user signs off on.**

Core rule: **detection is mechanical, deletion is a judgment call.** Never
delete on "no references found" alone. A symbol with zero internal callers may
still be a library's public API, a framework entry point, or reached
dynamically. When in doubt, surface it in the plan — do not delete it.

## Workflow

Phases 1–5 produce the review artifact (the removal plan). Phases 6–7 execute it
only after the user approves. Follow them in order; do not skip verification.

### 1. Establish a safety net

Before changing anything, confirm the working tree is clean and committed (or
stashed), so every removal is trivially revertible:

```bash
git status --short        # must be clean, or get user agreement to proceed
git rev-parse --abbrev-ref HEAD
```

If the repo is not under version control, stop and tell the user that this
skill relies on Git as the undo mechanism and ask them to initialise/commit
first. Do not proceed without a way to revert.

### 2. Detect the stack(s)

Identify which ecosystems are present by looking for manifest files at the repo
root and in subprojects (monorepos commonly mix several):

| Marker file(s) | Stack |
| --- | --- |
| `package.json`, `tsconfig.json` | JavaScript / TypeScript |
| `pyproject.toml`, `setup.py`, `requirements.txt` | Python |
| `go.mod` | Go |
| `Cargo.toml` | Rust |
| `pom.xml`, `build.gradle`, `*.gradle.kts` | Java / Kotlin (JVM) |
| `*.csproj`, `*.sln` | C# / .NET |
| `Gemfile`, `*.gemspec` | Ruby |

```bash
ls -a; find . -maxdepth 3 -name package.json -o -name go.mod -o -name Cargo.toml \
  -o -name pyproject.toml -o -name pom.xml 2>/dev/null
```

For each stack found, read the matching section of `references/detectors.md` for
the exact detector tools, install commands, and known gotchas. **Read that file
before running detectors**, it carries the per-language specifics this file
deliberately omits.

### 3. Run the detector(s)

Run the recommended detector for each stack (see `references/detectors.md`).
Prefer whole-project analysers (e.g. Knip, `deadcode`, `vulture`) over
per-file linters, because they understand cross-file reachability. Capture the
raw candidate list — do not act on it yet.

If a detector isn't installed, ask the user before installing it. If they
decline, fall back to the more conservative options noted in the references
(e.g. the language's built-in compiler/linter warnings) rather than guessing.

### 4. Classify candidates into confidence tiers

Sort every candidate into one of three tiers. This is the heart of the skill.

**Tier 1 — Safe to remove automatically.** Local, file-internal, and provably
unreachable:

- Unused local variables and parameters the linter flags
- Unused imports / `use` statements
- Unreachable code after `return`/`throw`/`break`
- Private functions/methods with zero references *within the same module* and no
  dynamic-reference risk (see filters below)

**Tier 2 — Remove only after a reference check.** Likely dead but worth
verifying:

- Module-internal functions/classes with no callers
- Whole files a whole-project analyser reports as unreferenced
- Symbols left behind by a reworked feature (the user's main case): grep the old
  symbol name across the repo *and* config files before deleting

**Tier 3 — Surface for human review, never auto-delete.** High false-positive
risk:

- Exported / public symbols (a library's API used by external consumers won't
  show internal callers)
- Framework entry points: route handlers, lifecycle hooks, event subscribers,
  CLI command registrations, serializers, migrations
- Anything referenced dynamically (see filters)
- Test-only helpers and fixtures
- Public config, plugin registrations, dependency-injection targets

When unsure which tier something belongs in, **promote it to a higher (safer)
tier.** Err toward keeping code.

### 5. Apply false-positive filters before deleting Tier 1/2

Even within the "safe" tiers, screen each candidate against dynamic-reference
patterns that static analysis misses. Search for the symbol name as a string:

```bash
rg -n --fixed-strings 'symbolName'        # any textual reference at all
```

Hold back (move to Tier 3) if the name appears in: string literals (reflection,
dynamic dispatch, `getattr`/`__import__`, `eval`), config/manifest files,
templates or markup, DI containers, serialization keys, public API barrels
(`index.ts`, `__init__.py`, `mod.rs`, `lib.rs`), or annotations/decorators that
register it with a framework.

### 6. Present the removal plan and get approval

This is the review artifact. Before deleting anything, report the tiered plan to
the user using the finding format below, and **wait for their go-ahead.** Tier 1
and Tier 2 items are proposed for removal; Tier 3 items are surfaced for their
decision and never removed on your initiative. The user may approve all, approve
only some tiers/items, or decline — proceed accordingly.

For each candidate (or coherent group), report:

```markdown
### [Tier N] <symbol / file / dependency>
- **Where:** <file:line or area>
- **Why it's flagged:** <detector signal + reachability evidence>
- **Risk if removed:** <none / low / unknown — e.g. "possible dynamic reference in config">
- **Action:** <remove in batch X | needs your decision because …>
```

End the plan with a summary: how many items per tier, what you propose to remove
now, what you're holding for review, and which detector hits you judged live
(noise) and why. Only after the user approves do you continue to step 7.

### 7. Remove in small, reviewable, verifiable batches

Never delete everything in one commit. For each batch:

1. Remove one coherent group (one tier, or one feature's leftovers).
2. Run the build/type-check and the test suite.
3. If green, commit with a clear message (e.g. `chore: remove dead X after Y rework`).
4. If red, revert that batch and move the items to Tier 3 for review.

```bash
# examples — use the project's actual commands
npm run build && npm test        # JS/TS
go build ./... && go test ./...   # Go
cargo build && cargo test         # Rust
pytest                            # Python
```

Removing dead code can cascade: deleting a function may make its private
helpers dead too. Re-run the detector after each batch and repeat from step 4
until the detector is quiet or only Tier 3 items remain.

### 8. Final report

Once removal is done, close the loop: what was actually removed (by tier and
batch, with commits), what was left for the user's review and why, and any
candidates the detector flagged that turned out to be live (so they know the
detector has noise). Restate outstanding Tier 3 items as a list they can decide
on later — do not delete them on your own initiative.

## Hard constraints

- **Plan before deletion.** Present the tiered removal plan and get the user's
  approval (step 6) before removing anything. No surprise deletions.
- **Git is the undo button.** No removal happens on a dirty/uncommitted tree
  without explicit user agreement.
- **No bulk deletion.** Small batches, each verified by build + tests.
- **Never delete a Tier 3 item autonomously.** Exported symbols, entry points,
  and dynamically-referenced code are surfaced, not removed.
- **Comments and commented-out code:** offer to remove commented-out *code*
  blocks (Git preserves history) but leave explanatory comments alone.
- **Don't touch generated files** (e.g. `*.pb.go`, `*_generated.*`, lockfiles) —
  fix the generator's input instead.

## Reference files

- `references/detectors.md` — per-ecosystem detector tools, install commands,
  recommended invocations, and language-specific false-positive gotchas. Read
  the relevant section in step 2/3 before running any detector.
