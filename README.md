# slop

A codebase linter with no code.

**slop.md** is a single document you drop into an LLM's context to detect and
clean accumulated cruft in any codebase: stale docs, dead code, ghost
dependencies, orphan files, debug residue, config drift.

No CLI. No install. No dependencies. The LLM is the runtime.

## Usage

Copy `slop.md` into your project, your LLM's system prompt, or your
conversation. Then:

```
Analyze this codebase using the slop.md spec. Focus on high-severity findings.
```

Works with any LLM, any language, any framework.

## Why

Traditional linters check syntax and style. They can't tell you that your
README describes a setup process you changed six months ago, that half your
package.json is packages nobody imports, or that a TODO from 2021 references a
Jira ticket that no longer exists.

Slop is fuzzy. Detecting it requires judgment. LLMs have judgment. Rules don't.

## What it catches

| Category | What it is |
|---|---|
| Phantom Docs | Docs that describe a system that no longer exists |
| Dead Code | Functions nobody calls, code in comments, unreachable branches |
| Ghost Dependencies | Packages in your manifest that nothing imports |
| Orphan Files | Files nothing references: dead configs, obsolete tests |
| Stale Markers | TODOs and FIXMEs that have outlived their context |
| Debug Residue | console.logs, hardcoded test data, leftover debugging |
| Structural Cruft | Empty dirs, inconsistent naming, pointless wrappers |
| Config Drift | .env.example, CI steps, and tool configs that don't match reality |

## Philosophy

Every codebase has two maps: what it *claims* to be (docs, config, manifests)
and what it *actually is* (imports, calls, runtime behavior). Slop lives in the
gaps between these two maps.

slop.md instructs the LLM to build a mental graph of both maps and find the
disconnections: dangling references, isolated nodes, contradictory edges. A
function isn't dead code because of something wrong with the function. It's dead
because nothing in the system points to it anymore.

Inspired by:
- Andrej Karpathy's [LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) -
  an "idea file" you paste into your LLM agent, and the agent builds out the specifics
- Drew Breunig's [whenwords](https://github.com/dbreunig/whenwords) -
  a [software library with no code](https://www.dbreunig.com/2026/01/08/a-software-library-with-no-code.html),
  just a spec the LLM implements
- [Excalibur](https://github.com/viemccoy/excalibur)'s hypergraph-of-thought
  reasoning - build a graph, find structural anomalies

The spec is the library. The LLM is the runtime. One markdown file.

## Integration ideas

- **Claude Code / Cursor / Copilot**: Add slop.md to your project root or
  .cursor/rules. The LLM picks it up automatically.
- **CI/CD**: Pipe your repo through an LLM with slop.md in context, fail the
  build on high-severity findings.
- **Code review**: Include slop.md in review context to catch cruft alongside
  logic bugs.
- **Scheduled audits**: Run weekly/monthly slop checks to track hygiene over
  time.

## FAQ

**Isn't this just asking an LLM to review code?**
Yes, with a structured taxonomy and consistent output format. The difference
between "review this code" and a slop audit is the difference between "look at
this" and a checklist. Structure produces consistency.

**What about false positives?**
slop.md instructs the LLM to be conservative. False positives are worse than
false negatives. A noisy report trains people to ignore it.

**Can I customize the categories?**
Yes. Tell the LLM to focus on specific categories, ignore others, add your own,
or adjust severity thresholds. The taxonomy is a starting point.

**Does this replace ESLint / ruff / clippy?**
No. Those catch syntax errors and style violations with deterministic precision.
slop.md catches the fuzzy stuff they can't: semantic drift between what your
codebase says and what it does.

## License

MIT
