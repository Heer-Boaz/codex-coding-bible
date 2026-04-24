# Quality Analysis Workflow

Use this when running, designing, or fixing quality rules.

## Command Discovery

- Prefer project-local scripts and dependencies.
- In JavaScript/TypeScript projects, inspect `package.json` for scripts named
  `analyze`, `analyze:*`, `lint`, `lint:*`, `quality`, `quality:*`, `check`,
  `check:*`, `typecheck`, `test`, and formatter checks.
- In native or mixed projects, inspect build files, task files, CI workflows,
  and documented commands.

## Rule Design

- A rule should detect code-quality or architecture debt, not formatting
  preference.
- Prefer precise AST, token, parser, or semantic logic over text grep when the
  language parser is available.
- Keep false positives low. A rule that forces worse code must be changed before
  product code is touched.
- Report the smallest meaningful construct.
- Preserve semantic targets in fingerprints.
- Avoid duplicate reports across exact duplicate, semantic duplicate, and
  normalized-body duplicate rules.
- Keep project-specific policy in explicit configuration.

## Fixing Existing Code

1. Run the relevant analyzer root.
2. Inspect sample findings before changing product code.
3. Decide whether each finding is real debt, a rule bug, or an intentional
   exception.
4. Fix analyzer bugs first.
5. Fix product code only when the resulting code is simpler, faster, or better
   owned.
6. Re-run the same analyzer root and compare counts.

## Suppressions

Suppressions must be local, rule-specific when possible, and include a reason.

```ts
// @code-quality disable-next-line empty_catch_pattern -- external cleanup callback may throw during teardown
try {
    cleanupExternalHandle();
} catch {
}
```

Allowed regions include hot-path, required-state, repeated-sequence-acceptable,
normalized-body-acceptable, value-or-boundary, fallible-boundary,
numeric-sanitization-acceptable, allocation-fallback-acceptable, and
optional-chain-acceptable. Region starts may carry local labels after the region
kind.

## Architecture Rules

- Rules should describe ownership, direction, layering, public contracts, and
  boundary crossings in terms the project can configure.
- A layer violation should point to the importing file, imported target, and
  violated direction.
- A public-contract violation should identify the private object or shortcut and
  name the declared boundary it should use.
- A hot-path rule should report allocations, normalization, parsing, closures, or
  string churn only where the path is actually hot or marked as such.

## Lua Analysis

- Use the Lua parser and scope model when available.
- Track module aliases, local `<const>` bindings, single-use locals, staged
  exports, global constant copies, table field labels, event names, and lifecycle
  hooks as semantic concepts.
- Keep runtime-profile and user-layer-profile differences explicit in rule
  configuration.
- Read `lua-quality-rules.md` before changing Lua analyzer behavior.

## C++ Analysis

- Use clang-tidy, cppcheck, compiler diagnostics, and custom token/AST rules as
  complementary signals.
- Keep compile database and header filters explicit when running clang-tidy.
- Custom C++ rules should operate on tokens, declarations, function ranges,
  includes, local bindings, call targets, and normalized bodies rather than text
  grep.
- Track analysis regions for hot paths, value-or-boundaries, numeric
  sanitization exceptions, optional fallback exceptions, and local suppressions.
- Read `cpp-quality-rules.md` before changing C++ analyzer behavior.

## Final Audit

Before finishing:

- Search your diff for `return;`, `?? null`, `typeof`, `?.`, `catch`, `ensure`,
  `fallback`, `provider`, `service`, `host`, `descriptor`, and new one-use
  locals.
- Check whether any new helper is genuinely shared or just ceremony.
- Check whether hot-path edits allocate arrays, objects, closures, strings, or
  repeated normalized values.
- Check whether any suppression is local, rule-specific, and honestly reasoned.
- Run `git diff --check`.
