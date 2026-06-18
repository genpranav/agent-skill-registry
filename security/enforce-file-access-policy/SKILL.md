---
name: enforce-file-access-policy
description: >-
  Configures and validates Claude Code's file-access permissions in
  .claude/settings.json to (1) restrict all tool access to the current
  repository root — blocking path traversal above it — and (2) deny Read,
  Edit, Write, Glob, and Grep on any file or directory whose name matches
  common secrets patterns (password, passwd, secret, credential, api-key, jwt,
  private-key, ssh-key, vault-token, and ~25 more variants; full list in
  references/sensitive-patterns.md). Use whenever the user wants to lock down
  what Claude can touch, set up project-level access guardrails, prevent Claude
  from reading keys or credentials, asks about permissions or access control in
  settings.json, or is worried about accidental secret exposure — even if they
  don't say "settings" or "deny rules" explicitly. If matching deny rules
  already exist, the skill validates each one exhaustively and reports gaps
  without overwriting anything.
version: 0.1.0
---

# Setup Project Access

This skill is a security-configuration lens. It writes or validates the
`permissions.deny` block in `.claude/settings.json` (or
`.claude/settings.local.json`) so that:

1. **Repository boundary** — file tools cannot traverse above the repo root via
   relative paths (`../`).
2. **Sensitive-name blocking** — `Read`, `Edit`, `Write`, `Glob`, and `Grep`
   are denied on any path whose name fragment matches the canonical list of
   secrets-related tokens (passwords, keys, credentials, tokens, and their
   common separator variants).

**detecting partial coverage**
(rules present but missing variants or tool verbs) and **verifying the final
configuration is airtight**. Never overwrite what is already there without
inspecting it first.

---

## When this applies

- User sets up a new project and wants Claude constrained to the repo.
- User asks Claude not to read secrets, credentials, or key files.
- User says "restrict file access", "lock down what you can touch", "don't read
  outside the project", or similar — even without mentioning settings.json.
- After any edit to `.claude/settings.json` that could have disturbed existing
  deny rules.

---

## Workflow

Follow these steps in order. Never skip validation (step 5).

### 1. Locate the settings file

Check in priority order:

```
.claude/settings.local.json   ← project-local, typically git-ignored
.claude/settings.json         ← project-shared, committed to the repo
```

If neither exists, note it and proceed to step 3 — the file will be created.

Before writing, determine which file to target:

- Prefer `.claude/settings.local.json` when the purpose is per-developer safety
  (the rules never reach the remote).
- Prefer `.claude/settings.json` when the user wants team-wide enforcement (the
  rules are committed).

**Ask the user which they prefer before writing.** Show the target path in your
question.

Also check `.gitignore`: if the user chose `settings.local.json` but it is not
git-ignored, warn them that the "local" file will be committed as-is.

### 2. Parse and audit existing permissions

Read the target file. Look for `permissions.deny`. For every rule found:

- Confirm it is syntactically valid (parseable as a JSON string, balanced
  parentheses, no stray characters).
- Confirm the tool verb is one of the recognised values: `Read`, `Edit`,
  `Write`, `Glob`, `Grep` (case-sensitive, exact match).
- Confirm the glob suffix matches the expected pattern `**/*<fragment>*`.
- Record which (fragment × tool-verb) pairs are already covered and which are
  missing.

Also inspect `permissions.allow`: an allow rule that whitelists a path covered
by a deny rule silently wins. Flag any such conflict — do not silently delete
the allow rule.

### 3. Build the required deny-rule set

The complete set of required deny rules is enumerated in
`references/sensitive-patterns.md`. Read that file now — it contains the
canonical JSON array you will diff against the existing rules.

The required rules cover two groups:

**Group A — Repository boundary (5 rules):**

```json
"Read(../**)",
"Edit(../**)",
"Write(../**)",
"Glob(../**)",
"Grep(../**)"
```

These block relative-path traversal above the repo root. Note: they do not
block absolute paths outside the repo (e.g. `/etc/passwd`). If the user needs
that level of isolation, note the limitation and recommend running Claude Code
in a sandboxed container.

**Group B — Sensitive-name blocking:**

For each sensitive name fragment, five rules are required (one per tool verb):

```
Read(**/*<fragment>*),
Edit(**/*<fragment>*),
Write(**/*<fragment>*),
Glob(**/*<fragment>*),
Grep(**/*<fragment>*)
```

The full fragment list is in `references/sensitive-patterns.md`. Consult it —
do not rely on memory for the complete set.

### 4. Determine the action

Compare required rules against what is already present:

| Situation | Action |
|---|---|
| No settings file | Create file with full deny-rule set (after confirmation). |
| File exists, no `permissions` key | Add `permissions.deny` array (after confirmation). |
| File exists, `permissions.deny` missing | Add the deny array, preserve rest of file (after confirmation). |
| File exists, deny present but incomplete | Append only the missing rules (after confirmation). |
| File exists, deny present and complete | **Validate only — do not write anything.** |

Never delete or rewrite existing rules. Only add what is missing.

### 5. Write (if needed) — confirm first

Before touching any file, show the user the exact diff: the JSON strings you
intend to add. Wait for an explicit yes. If the user says no, skip writing and
proceed to validation against the current state.

When merging into an existing file, parse the JSON, insert into
`permissions.deny`, and re-serialise — do not use string concatenation or
regex replacement on JSON.

### 6. Validate the final state

Re-read the settings file after writing (or immediately if no write was needed)
and run the full validation checklist. Report each item as `PASS`, `FAIL`, or
`WARN`.

**Structural checks:**

- [ ] `.claude/` directory exists at repo root.
- [ ] Target settings file exists and is parseable JSON.
- [ ] `permissions` key present.
- [ ] `permissions.deny` is a non-empty array.
- [ ] All existing rule strings have balanced parentheses and no stray
  characters.

**Repository-boundary checks (Group A):**

- [ ] `Read(../**)` present.
- [ ] `Edit(../**)` present.
- [ ] `Write(../**)` present.
- [ ] `Glob(../**)` present.
- [ ] `Grep(../**)` present.

**Sensitive-name checks (Group B):**

For every fragment in `references/sensitive-patterns.md`, confirm all five
tool-verb rules are present. List each missing (fragment × verb) pair as a
`FAIL`. A fragment with all five verbs covered is one `PASS` line — do not
produce fifty individual PASS lines for correct entries, keep the report
readable.

**Conflict check:**

- [ ] No `permissions.allow` rule whitelists a path that a deny rule targets.

### 7. Report

Use this structure:

```markdown
## Access-control setup — <PASS | PARTIAL | FAIL>

**Settings file:** `<path>`

### Validation results

#### Structural
- [PASS] `.claude/` directory present
- [PASS] `settings.json` is valid JSON
- …

#### Repository boundary
- [PASS] All five `../` deny rules present

#### Sensitive-name coverage
- [PASS] password, passwords, passwd — all 5 tool verbs covered
- [FAIL] jwt — missing `Glob` and `Grep` rules
- …

#### Conflict check
- [PASS] No allow rules conflict with deny rules

### Rules written (if any)
<list the exact JSON strings added>

### Remaining gaps (if any)
<list the exact JSON strings still missing, ready to paste>
```

Final verdict:

- **PASS** — all rules present and valid; nothing to do.
- **PARTIAL** — some rules present; gaps listed; re-run after adding them.
- **FAIL** — configuration absent or critically broken; rules written (or user
  declined).

---

## Hard constraints

- **Inspect before writing.** Read the existing file and report what you found
  before proposing any changes.
- **Confirm before writing.** Show the exact diff and wait for yes.
- **Never delete** an existing rule — it may be intentional. Only add.
- **Never duplicate** a rule that already exists verbatim.
- **No string surgery on JSON.** Parse the file, mutate the object, re-serialise.
- **Always validate** (step 6) regardless of whether you wrote anything.
- **Note the absolute-path gap.** Rules using `../` block relative traversal
  but not absolute paths. Say so explicitly in the report if it is relevant.

## Reference files

- `references/sensitive-patterns.md` — complete canonical list of sensitive
  name fragments with all separator variants, and the full required JSON array.
  Read this in step 3 before building the diff.
