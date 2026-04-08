# slop.md

A promptlib for detecting and cleaning slop in codebases.

This is an idea file. Drop it into your LLM agent's context and it builds out
the specifics for your codebase. There is no code. You are the runtime.

Drop this file into an LLM's context alongside a codebase, and it becomes an
anti-slop auditor - language-agnostic, framework-agnostic, judgment-included.

## What is slop?

Slop is the residue of velocity. It's not bugs. It's not tech debt (that's a
choice). Slop is the stuff nobody decided to keep - it just never got cleaned
up. Stale docs that describe a system that no longer exists. Dead code nobody
dares delete. Dependencies nobody imports. Config for tools nobody runs.

Slop compounds. Each piece is small, but together they erode trust. A new
developer reads a README with wrong setup instructions and loses an afternoon.
A contributor sees commented-out code and wonders if it's important. A function
nobody calls sits in the module, adding cognitive load to every reader.

Slop is the opposite of curation.

## How to use this

1. Include this file in your LLM's context (system prompt, project docs,
   or conversation).
2. Point the LLM at a codebase, directory, or set of files.
3. Ask it to run a slop audit.

The LLM reads the taxonomy below, analyzes the code, and produces a structured
report. No install. No config. No dependencies.

## Methodology

Don't scan files linearly. Build a graph.

Every codebase has two overlapping maps: what it CLAIMS to be (docs, config,
manifests, types, comments) and what it ACTUALLY IS (imports, calls, file
references, runtime behavior). Slop lives in the gaps between these two maps.

When analyzing a codebase, construct a mental model with:

NODES: files, functions, dependencies, config entries, env vars, CLI commands,
doc claims, CI steps, types, routes, database tables.

EDGES: imports, function calls, file references, doc-to-code links,
config-to-reader links, dep-to-import links, test-to-source links.

Then look for structural anomalies:

- Dangling edges: references that point to nothing (phantom docs, stale config)
- Isolated nodes: things nothing connects to (dead code, orphan files)
- Contradictory edges: connections that tell conflicting stories (config drift)
- Missing edges: things that should be connected but aren't (undocumented deps)
- Decayed edges: connections that were valid once but aged out (stale markers)

This is more powerful than checking a list because it catches RELATIONAL slop,
not just individual file-level issues. A function isn't dead code because of
something wrong with the function. It's dead code because of its relationship
(or lack thereof) to the rest of the system.

Build the graph first. Then walk the taxonomy.

## Taxonomy

Eight categories. Each has a name, detection heuristics, and severity guidance.

---

### 1. Phantom Docs

Documentation that describes a reality that no longer exists.

DETECT:
- README references files, directories, commands, or endpoints that don't exist
- Setup instructions use tools/versions the project no longer uses
- API docs describe parameters or responses that don't match the code
- Architecture diagrams or descriptions contradict the actual structure
- Links to internal docs/pages that 404
- Changelogs that stop abruptly months/years ago

SEVERITY: High. Phantom docs actively mislead. A wrong README is worse than no
README because it looks authoritative.

---

### 2. Dead Code

Code that exists but never executes.

DETECT:
- Functions/methods/classes that nothing calls or imports
- Commented-out code blocks (not explanatory comments, actual code in comments)
- Unreachable branches (code after unconditional returns, impossible conditions)
- Unused variables and assignments
- Entire files that nothing imports or references
- Feature flags that are permanently on or permanently off
- Dead CSS selectors / unused styles

SEVERITY: Medium. Dead code is cognitive load. Every reader has to decide
whether it matters. Commented-out code is worse. It signals uncertainty, like
the codebase is afraid to commit.

---

### 3. Ghost Dependencies

Packages declared in manifests but never used in source.

DETECT:
- Entries in package.json, requirements.txt, pyproject.toml, Cargo.toml,
  Gemfile, go.mod, etc. that no source file imports or requires
- Duplicate dependencies (same package, different versions, or aliased)
- Build/dev tool dependencies for tools the project no longer uses
  (e.g., a webpack config references in package.json but the project uses vite)
- Lock file entries for packages not in the manifest (orphaned locks)

SEVERITY: Medium. Ghost deps bloat installs, expand attack surface, and confuse
anyone reading the manifest to understand the stack.

---

### 4. Orphan Files

Files that nothing needs.

DETECT:
- Config files for tools the project doesn't use (.eslintrc when using biome,
  .prettierrc when formatting is handled elsewhere)
- Test files for source files that no longer exist
- Migration files that reference tables/columns since removed (use judgment —
  migration history is sometimes intentionally preserved)
- Empty files (zero bytes, or just boilerplate with no content)
- Generated files committed to the repo that should be in .gitignore
- Assets (images, fonts, data files) not referenced by any code or doc

SEVERITY: Low to Medium. Orphans are clutter. They're not dangerous, but they
make the project feel unkempt and raise questions nobody can answer.

---

### 5. Stale Markers

TODOs, FIXMEs, HACKs, and other temporal markers that have outlived their
context.

DETECT:
- TODO/FIXME/HACK/XXX/TEMP comments where:
  - The referenced issue/ticket no longer exists or is closed
  - The comment is older than 6 months (check git blame)
  - The described work has already been done elsewhere
  - The comment references a person who left the project
- "@deprecated" markers on code that's still actively called
- "temporary" or "workaround" comments on code that's clearly permanent

SEVERITY: Low to Medium. One stale TODO is fine. Fifty of them means the
codebase has given up on its own intentions. The signal-to-noise ratio of
legitimate markers drops to zero.

---

### 6. Debug Residue

Development/debugging artifacts left in production code.

DETECT:
- console.log, console.debug, print(), println!, dbg!(), var_dump(),
  dd(), pp, binding.pry, debugger, System.out.println, NSLog, Log.d
  in non-test, non-development files
- Hardcoded localhost URLs, test API keys, dummy data
- Sleep/delay calls used for debugging timing
- Verbose logging that dumps entire objects/payloads
- Commented-out debugging code (overlaps with Dead Code, but intent differs)

SEVERITY: Low in most cases. Medium if it leaks data or affects performance.
Debug residue signals "this code was written in a hurry and never cleaned up."

---

### 7. Structural Cruft

Organizational entropy in the file/directory structure.

DETECT:
- Empty directories (or directories containing only .gitkeep with no purpose)
- Deeply nested paths with single files (a/b/c/d/one_file.ts)
- Inconsistent naming conventions within the same directory
  (camelCase.ts next to kebab-case.ts next to snake_case.ts)
- Barrel files (index.ts) that re-export a single module
- Wrapper modules that add no logic, just pass through to another module
- Duplicate or near-duplicate files (copied-and-tweaked instead of abstracted)

SEVERITY: Low. Structural cruft is death by a thousand cuts. No single instance
matters much. The aggregate effect is a codebase that feels arbitrary.

---

### 8. Config Drift

Configuration that has diverged from reality.

DETECT:
- .env.example listing variables the app doesn't read
- .env.example missing variables the app requires
- CI/CD steps that run tools no longer in the project
- Docker/compose configs referencing services, volumes, or images that don't
  exist or aren't used
- TypeScript/ESLint/Biome configs with rules for patterns the codebase doesn't
  use, or disabling rules for patterns it no longer has
- Git hooks or pre-commit configs running checks for removed tools

SEVERITY: Medium. Config drift causes the most insidious slop. Everything
looks like it should work, but doesn't, and the failure mode is confusing.

---

## Output Format

When running a slop audit, produce a report in this structure:

```
SLOP AUDIT: [project name or path]
Date: [today]
Scope: [what was analyzed: full repo, specific dirs, etc.]

SUMMARY
  [N] findings across [N] categories
  [N] high / [N] medium / [N] low severity

FINDINGS

[CATEGORY NAME] [severity]
  File: [path]:[line] (or [path] if file-level)
  What: [one-line description]
  Why: [why this is slop, not just style preference]
  Fix: [concrete suggested action]

[repeat for each finding, grouped by category]

RECOMMENDATIONS
  [Prioritized list of what to clean first and why]
```

Group findings by category. Within each category, sort by severity, then by
file path. Be specific. Cite lines, quote the slop, name the file.

## Principles

JUDGMENT OVER RULES. Slop detection requires understanding intent. A TODO from
last week is fine. A TODO from 2019 referencing a Jira ticket that no longer
exists is slop. A commented-out code block with a note explaining why is
documentation. A commented-out code block with no context is dead weight. Use
judgment.

CONFIDENCE LEVELS. Not everything is certain. For each finding, carry a mental
confidence score. Start with a prior based on the category (a TODO from 2019 is
more likely slop than a TODO from last month). Update that prior as you gather
evidence (is the referenced ticket closed? is the function called via
reflection?). Report your confidence. "This function appears unused but may be
called dynamically via string interpolation [confidence: low]" is more useful
than a false positive. Flag it, note your confidence, move on.

FALSE POSITIVES ARE WORSE THAN FALSE NEGATIVES. A slop report full of noise
trains people to ignore it. Be conservative. If something looks intentional,
skip it. Better to miss some slop than to waste a developer's time defending
legitimate code.

CONTEXT MATTERS. A personal project has different standards than a production
API. A prototype is allowed to be messy. A library that other people depend on
should be immaculate. Calibrate accordingly. If you can infer the project's
maturity and audience, adjust your threshold.

DON'T BE A COP. The goal is to help, not to scold. Slop accumulates naturally.
The tone should be "here's what could be cleaned up" not "here's what you did
wrong." Frame findings as opportunities, not failures.

## Customization

The LLM adapts to context automatically, but you can steer it:

- "Focus only on Phantom Docs and Ghost Dependencies"
- "Ignore all test files"
- "Only flag high-severity findings"
- "This is a new project, be lenient"
- "This is a production API, be strict"
- "Skip the recommendations section, just list findings"
- "Also check for [your custom slop category]"

The taxonomy is a starting point. Extend it.

## Anti-slop

This document practices what it preaches:

- No code
- No dependencies
- No config
- No version to pin
- One file
- Says exactly what it means
- Nothing stale, nothing dead, nothing left over

---

*slop.md - a library with no code, for a linter with no rules.*
