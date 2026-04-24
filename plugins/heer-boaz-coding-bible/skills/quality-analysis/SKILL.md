---
name: quality-analysis
description: "Use when running, creating, fixing, or interpreting code-quality analyzers, lint rules, Lua quality rules, C++ quality rules, architecture checks, duplicate-code detectors, suppression audits, or quality reports in any project. Treat analyzer code as production code and fix rule precision before changing product code for noisy findings."
---

# Quality Analysis

Use this for analyzer/linter work and for quality reports that enforce coding
or architecture standards.

## Workflow

Read `../../references/quality-analysis-workflow.md` first. For architectural
judgment, also read `../../references/coding-architecture-rules.md`.
For Lua findings, read `../../references/lua-quality-rules.md`.
For C++ findings, read `../../references/cpp-quality-rules.md`.

1. Discover the project-local analysis entrypoint from package scripts, task
   files, build files, or documented commands.
2. Run the narrowest relevant analyzer root first.
3. Inspect representative findings before editing product code.
4. Classify each finding as real debt, analyzer bug, or intentional exception.
5. Fix analyzer bugs before product code.
6. Fix product code only when the result is simpler, faster, better owned, or
   more faithful to the architecture contract.
7. Re-run the same analysis root and compare counts.

## Rule Design

Prefer parser, AST, token, or semantic APIs over text matching when available.
Report the smallest meaningful construct and preserve the semantic identity of
the target. Avoid duplicate reports across exact duplicate, semantic duplicate,
and normalized-body rules.

Do not encode one project symbol, path, or name as a generic exception. If local
configuration is needed, make it explicit data and keep the rule itself honest.

## Suppressions

Suppressions must be local, rule-specific when possible, and include a reason.
They are allowed only for true boundaries, generated code, platform quirks, or
temporarily unavoidable debt that is named honestly.

Do not use broad suppressions to make a report green. A clean analyzer run means
the code is cleaner, not merely quieter.

## Final Audit

Before finishing, audit new analyzer code and touched product code for hidden
fallbacks, defensive initialization, new indirection, broad skips, repeated
semantic work, hot-path allocation, and formatting-only noise. Always run
`git diff --check`.
