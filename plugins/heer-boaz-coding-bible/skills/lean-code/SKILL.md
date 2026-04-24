---
name: lean-code
description: "Use when reviewing, cleaning up, refactoring, redesigning, simplifying, or fixing architecture and code-quality debt in any project, including Lua or C++ quality debt, and when feature work is blocked by bad code. This is a hard-stop cleanup gate: remove the named disease without adding wrappers, lazy initialization, defensive fallbacks, suppressions, compatibility aliases, or ownership-hiding layers."
---

# Lean Code

Use this for cleanup, refactor, architecture repair, code review, analyzer work,
and feature tasks blocked by bad code. This is not a general feature workflow.
It exists to stop cleanup work from adding more junk.

## Gate

Before editing, name the specific disease being removed. If the patch does not
remove a real disease, stop.

Good cleanup deletes, inlines, collapses, or moves ownership closer to the data.
Bad cleanup adds indirection, comments, tags, files, or abstractions while the
code remains harder to understand.

## Stop Immediately

Stop before editing if the likely patch needs:

- temporary architecture;
- facade, host, provider, service, descriptor, adapter, manager, or broker layers;
- callback injection to avoid naming the real owner;
- wrapper-only functions;
- lazy initialization in steady-state paths;
- defensive fallbacks around internal state that should exist;
- shadow work on a hot path that could update the real backend/device state;
- compatibility aliases or legacy fallback paths;
- analyzer skip lists, name/path/usage exemptions, or broad suppressions;
- more comments or tags than actual simplification.

Report the stop gate and the owner that must be understood first.

## Required Process

1. Read nearby code and `../../references/coding-architecture-rules.md`.
2. Search with `rg` for owners, state, helpers, boundaries, and counterparts.
3. Read `../../references/lua-quality-rules.md` for Lua sources.
4. Read `../../references/cpp-quality-rules.md` for C++ sources.
5. Use `../../references/reference-implementation-guide.md` for unfamiliar domains.
6. Make the smallest slice that removes the named disease now.
7. Audit the diff against `../../references/anti-patterns.md`.

## Analyzer And Tags

Analyzer code is production code. Do not hide debt by names, paths, usage
counts, generic skips, or broad suppressions. Fix rules so bad code is exposed
precisely.

If code must be tagged or suppressed, the local comment must say why that code
is still bad, exceptional, or an unavoidable boundary. A tag is not a certificate
of quality.

For rule work, use `quality-analysis` and read
`../../references/quality-analysis-workflow.md`.

## Validation

Use the existing project build, test, analyzer, and validation entrypoints for
the touched area. Add narrower runs only when they increase confidence. Always
run `git diff --check` before finishing.
