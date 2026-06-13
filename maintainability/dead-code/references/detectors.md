# Dead Code Detectors by Ecosystem

Per-language detector tools, install commands, recommended invocations, and the
false-positive gotchas specific to each. Read the section matching the stack
detected in step 2 of `SKILL.md` before running anything.

## Contents

- [JavaScript / TypeScript](#javascript--typescript)
- [Python](#python)
- [Go](#go)
- [Rust](#rust)
- [Java / Kotlin (JVM)](#java--kotlin-jvm)
- [C# / .NET](#c--net)
- [Ruby](#ruby)
- [Last-resort, language-agnostic](#last-resort-language-agnostic)

General notes:
- Prefer whole-project analysers over per-file linters — they reason about
  cross-file reachability, which is what "dead code" really means.
- Run detectors from the directory containing the manifest file.
- "Unused dependency" detection is separate from "unused code" detection; both
  are listed where a good tool exists.
- Every tool here produces false positives on dynamic references and public
  API. Treat all output as *candidates*, then apply the tiering in SKILL.md.

---

## JavaScript / TypeScript

**Recommended (whole-project): Knip.** Finds unused files, exports, dependencies,
and members across the project graph.

```bash
npx knip                      # report
npx knip --include files,exports,dependencies
```

**Unused exports only: ts-prune** (lighter, TS-specific):

```bash
npx ts-prune
```

**Per-file unused locals/imports: ESLint:**

```bash
npx eslint . --rule '{"no-unused-vars":"error"}'
# TS projects: @typescript-eslint/no-unused-vars
```

**Unused dependencies: depcheck:**

```bash
npx depcheck
```

Gotchas:
- Knip honours `package.json` `exports`/`bin` and common framework entry points,
  but custom dynamic `import()` by computed string evades it — keep those.
- Anything re-exported through a barrel (`index.ts`) often shows as "unused
  export" while being part of the public API. Tier 3.
- Type-only exports consumed by downstream packages look unused locally.

---

## Python

**Recommended: vulture.** Reports unused functions, classes, variables, imports.

```bash
pip install vulture
vulture path/to/pkg            # raise/lower --min-confidence to tune noise
vulture path/to/pkg --min-confidence 80
```

**Unused imports (fast, precise): Ruff** (`F401` and related):

```bash
pip install ruff
ruff check --select F401,F811,F841 .
```

Gotchas:
- Dynamic access via `getattr`, `__import__`, `importlib`, and string-based
  plugin registries is invisible to static tools — vulture has a whitelist
  mechanism; use it for known dynamic names.
- Names in `__all__` and re-exported in `__init__.py` are public API → Tier 3.
- Pytest fixtures, Django/Flask view functions, Celery tasks, and click/argparse
  commands are entry points reached by the framework, not by callers → Tier 3.

---

## Go

**Recommended: `deadcode`** (official, from `golang.org/x/tools`). Reachability
analysis from `main`:

```bash
go install golang.org/x/tools/cmd/deadcode@latest
deadcode ./...
```

**Also useful: staticcheck** (`U1000` unused code) and the compiler itself
(unused imports/locals are hard errors in Go):

```bash
go install honnef.co/go/tools/cmd/staticcheck@latest
staticcheck ./...
go build ./...                 # fails on unused imports / locals
```

Gotchas:
- `deadcode` reasons from entry points; code reached only via `reflect`,
  `go:linkname`, or build tags for another OS/arch may be falsely flagged.
- Exported identifiers consumed by other modules won't show as used in a library
  → Tier 3.
- `init()` functions and `//go:generate` targets are entry points.

---

## Rust

**Built-in first.** The compiler already warns on dead code:

```bash
cargo build 2>&1 | grep -A2 'never used'
# do NOT rely on #![allow(dead_code)] being absent — check for it
```

**Unused dependencies: cargo-machete** (fast) or `cargo-udeps` (precise, nightly):

```bash
cargo install cargo-machete && cargo machete
cargo install cargo-udeps && cargo +nightly udeps
```

Gotchas:
- `pub` items in a library crate are the public API; the compiler won't warn on
  them even if nothing internal uses them → Tier 3.
- `#[no_mangle]`, `#[used]`, FFI exports, and items behind `#[cfg(...)]` for
  other targets are reachable externally.
- Macro-generated and trait-impl methods can look unused.

---

## Java / Kotlin (JVM)

No single clean CLI equivalent of Knip; lean on static analysers.

```bash
# PMD: unused private fields/methods/locals
pmd check -d src -R category/java/bestpractices.xml -f text
# SpotBugs and IDE inspections (IntelliJ "Unused declaration") also work
```

Gotchas — JVM dead-code detection is especially noisy because of:
- Reflection, dependency injection (Spring `@Component`/`@Autowired`), and
  bean wiring by name — heavily dynamic. Treat DI-managed classes as Tier 3.
- Public API of a library, JPA entities, serialized classes, and JNI methods.
- Annotation-driven entry points (`@Test`, `@Scheduled`, `@EventListener`,
  servlet/controller mappings).
Restrict confident removal to `private` members the analyser flags, and surface
everything else.

---

## C# / .NET

**Built-in Roslyn analyzers** report unused members (`IDE0051`), unused private
fields (`IDE0052`), unused usings (`IDE0005`), unused parameters (`IDE0060`):

```bash
dotnet build /warnaserror:IDE0051,IDE0052
dotnet format analyzers --verify-no-changes   # surfaces fixable diagnostics
```

Gotchas:
- Reflection, DI (`IServiceCollection`), and serialization (`System.Text.Json`,
  data contracts) reach types dynamically → Tier 3.
- Controllers, Razor page handlers, and `[Fact]`/`[Theory]` test methods are
  framework entry points.
- `public` API of a class library won't be flagged.

---

## Ruby

```bash
gem install debride && debride .       # likely-unused methods
# RuboCop for unused locals/assignments:
rubocop --only Lint/UselessAssignment,Lint/UnusedMethodArgument
```

Gotchas:
- Ruby is extremely dynamic — `send`, `method_missing`, `define_method`, and
  metaprogramming defeat static analysis. debride is best-effort; verify with
  tests aggressively.
- Rails controller actions, callbacks, and routes are framework-reached.

---

## Last-resort, language-agnostic

When no proper detector exists for a stack, or to corroborate a candidate,
count textual references with ripgrep. This is **weak evidence** — it cannot see
reachability and over-counts comments/strings — so only ever use it to *keep*
code (a hit means "do not delete"), never as sole grounds to delete.

```bash
rg -n --fixed-strings 'symbolName'           # all references anywhere
rg -n --fixed-strings -g '!*test*' 'symbolName'   # excluding tests
rg -nP '\bsymbolName\b'                       # word-boundary matches
```

Always also grep config, template, and manifest files, since dynamic wiring
lives there:

```bash
rg -n --fixed-strings 'symbolName' -g '*.{json,yaml,yml,toml,xml,env,html}'
```

A candidate with zero textual references *anywhere* (code, config, templates)
across the whole repo is the strongest signal you'll get without a real
analyser — but still verify with a build + test run before committing the
removal.
