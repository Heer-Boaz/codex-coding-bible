# C++ Quality Rules

These rules apply to C++ runtime, platform, native, and tooling code. They align
with the shared TypeScript quality rules while accounting for C++ ownership,
headers, optional values, RAII, static analysis, and hot paths.

## Ownership And Boundaries

- Put behavior at the owner of the data, lifecycle, and protocol.
- Do not add pass-through managers, adapters, brokers, providers, or facade
  modules to avoid touching the real owner.
- Use RAII and direct object ownership for resources. Prefer references, spans,
  string views, handles, and explicit ownership transfer over nullable global
  state or hidden lifetime side tables.
- Use pointers and `std::optional` only for genuinely optional state.
- External boundaries may validate and translate. Internal code should consume
  already-valid contract values.

## Headers And Includes

- Includes express dependency direction. Lower layers must not include upward
  feature layers for convenience.
- Prefer the smallest owner-facing header that exposes the needed contract.
- Keep headers free of convenience aliases that hide ownership or pull broad
  dependency graphs into unrelated translation units.
- Include ordering and self-sufficient headers should reveal missing includes,
  not mask them through incidental prior includes.

## Optional, Null, And Fallbacks

- `std::optional::value_or(...)` eagerly evaluates its fallback. Use an explicit
  branch when the fallback allocates, computes, queries, logs, or mutates.
- Internal `value_or` fallbacks are allowed only at an explicit value-or-boundary.
- Do not normalize missing values to `nullptr`, empty strings, empty containers,
  sentinel strings, or default objects just to keep going.
- Do not allocate fallback containers through nullish/default paths.

## Numeric Contracts

- Bound coordinates, cycles, sizes, indexes, and layout values once at their
  owner or input boundary.
- Repeated `clamp`, `min`, `max`, `floor`, `ceil`, `round`, `trunc`,
  `isfinite`, or equivalent checks are debt unless the line is an explicit
  numeric boundary.
- Hot paths must not defensively sanitize internal numeric contract values.
- Prefer domain-specific units and types when they prevent repeated conversion
  or sanitization work.

## Strings And Text

- Do not use empty strings as fallback/default state when missing state should be
  represented explicitly.
- Newline normalization belongs only at IO/text boundary regions.
- Repeated trim, split, join, case conversion, substring, replace, contains, or
  starts/ends checks usually means the code is missing a normalized concept.
- Replace repeated string OR-comparison chains with `switch`, enum-like
  dispatch, table/set membership, or a named parser/normalizer.

## Locals And Wrappers

- Remove single-use local aliases of short expressions unless the local names a
  real concept, narrows a type, captures a lifetime, or avoids repeated work.
- Use `const`/`constexpr` for meaningful local values that are read repeatedly
  and never reassigned.
- Do not add single-line wrapper functions or methods.
- Avoid single-property option/config structs.
- Do not add getter/setter or lookup aliases merely to satisfy another rule.

## Exceptions And Error Boundaries

- Empty catch blocks are forbidden. Catch only when you can handle, translate, or
  rethrow with useful boundary context.
- A catch that only rethrows is a useless wrapper.
- Catch-and-return fallback patterns hide internal contract failures.
- Terminal `return;` padding is no-op noise.

## Hot Paths

- Mark hot paths explicitly and treat the marker as a contract.
- Hot-path calls must not allocate capturing lambdas, `std::function`,
  `std::string`, `std::vector`, `std::map`, `std::unordered_map`,
  `std::optional`, or other temporary containers/payloads.
- Pass primitives, references, spans/views, retained structs, or scratch storage
  through hot paths.
- Precompute parser, lookup, normalization, and dispatch work that would repeat
  every frame, tick, packet, sample, pixel, or input event.

## Duplication And Facades

- Consecutive duplicate statements are forbidden.
- Repeated statement sequences should collapse into shared ownership or a single
  lifecycle operation.
- Normalized or semantically equivalent function bodies with different names are
  copied logic. Extract the real owner instead of multiplying variants.
- Analyzer rule files must own detection logic. Thin report wrappers and empty
  rule files are not quality work.

## Static Analysis Workflow

- Run project-local C++ checks first. Typical signals include compiler
  diagnostics, clang-tidy, cppcheck, custom architecture rules, duplicate
  detection, and hot-path rules.
- Tool findings are inputs to ownership judgment, not a replacement for it.
- Custom rules should use token/declaration/function/include information.

## Suppressions

- Suppressions must be local, rule-specific, and reasoned.
- Prefer an analysis region when a boundary is intentionally fallible, numeric,
  value-or, or hot-path-specific.
- Do not add path, name, usage-count, or tool-wide exceptions to make reports
  quiet.
