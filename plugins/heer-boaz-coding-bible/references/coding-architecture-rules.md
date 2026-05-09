# Coding And Architecture Rules

Use this as the doctrine for implementation, review, cleanup, and analysis in
any codebase.

## Architecture Identity

- Understand the project structure before editing. Identify the module that owns
  the state, behavior, public contract, and validation surface.
- Public runtime-visible behavior must flow through declared APIs, protocol
  boundaries, memory/register models, device models, command surfaces, or other
  explicit contracts already used by the project. Do not turn platform shortcuts
  or private backend handles into hidden public contracts.
- Split by real ownership, not by generic layers. A module should own a concept,
  data shape, lifecycle, or boundary that exists in the domain.
- Keep parallel implementations aligned conceptually. Divergence should come
  from language/runtime constraints, not accidental duplicate designs.

## Mirrored Runtime Parity

When a project has mirrored runtimes, such as TypeScript and C++, parity is a
hard implementation contract, not a naming exercise. This is the
mirrored-runtime parity gate for implementation, cleanup, analysis, and review
work.

- Mirrored runtime files must use the same relative path and basename unless an
  explicit exclusion names why the file has no counterpart.
- Public constants, functions, classes, and methods must match per mirrored file
  pair unless an explicit exclusion names the language/runtime reason.
- Runtime representation and ownership shape must match at boundaries. Do not
  create fake, stub, type-only, or cosmetically mirrored files.
- Before editing, identify the mirrored counterpart for every touched runtime
  file. Do not add, delete, rename, or materially change one side without the
  matching change or exclusion on the other side.
- Run the project-local parity audit when it exists. Do not claim parity unless
  the audit passes for the touched scope.
- After editing, start a Codex subagent as an independent reviewer. The reviewer
  must return a blocker for enterprise-style code, including GC-churn, heap
  allocation, indirection, defensive clutter, exception-control flow, or fake
  ownership in runtime paths unless a concrete construction, parser, decoder,
  IO, async, or ownership-transfer boundary is explicit. This is a hard fail,
  not an advisory warning.
- If Codex subagents cannot be started because the current environment or policy
  does not expose that capability, the gate has not passed. Say that explicitly,
  run the blocker checklist locally, and do not claim mirrored runtime parity.

Start a Codex subagent with this review prompt:

```text
Review this diff against all Coding Bible code and architecture rules. Read and
apply the relevant plugin references, especially coding-architecture-rules.md,
anti-patterns.md, and the language-specific rules for touched Lua or C++ files.

Hard rules:
- every code or architecture rule violation is a blocker
- wrong ownership, hidden public contracts, facade/provider/adapter layers,
  wrapper-only APIs, compatibility fallbacks, defensive clutter, null
  normalization, broad suppressions, or fake architecture are blockers
- single-call forwarding wrappers and one-line constant-binding range helpers
  are blockers unless they name a real domain contract outside runtime hot paths
- enterprise-style code is a blocker: reject object envelopes for scalar state,
  generic option bags, callback plumbing layers, telemetry/readiness wrappers,
  manager/facade/provider/service/adapter/broker scaffolding, defensive
  "just in case" branches, null-normalization, catch-and-continue fallbacks, and
  exception-shaped control flow unless the diff proves a concrete external
  boundary requires them
- exception throwing is a blocker in emulator-visible runtime/device/memory
  control flow; exceptions are allowed only for impossible internal bugs,
  construction/configuration failures, tests, tooling, or true external
  boundaries such as parsers, OS/browser APIs, decoders, network, and file IO
- performance regressions, repeated work, GC-churn, or heap allocation in
  emulator/runtime/device code are blockers unless the diff proves a concrete
  construction, parser, decoder, IO, async, or ownership-transfer boundary
- serialization, persistence, state replay, or runtime-boundary drift is a
  blocker
- same relative path and basename for mirrored runtime files
- same public constants/functions/classes/methods per file pair
- same runtime representation and ownership shape
- no fake/stub/type-only mirror files
- exclusions must be explicit and justified

Return blockers only.
```

Do not present a local checklist pass as an independent review.

## Feature Work

- Add behavior at the owner. Avoid callback injection, wrapper APIs, and new
  orchestration layers that only dodge ownership.
- Preserve direct control flow in performance-sensitive and state-sensitive code.
- Consider persistence, serialization, state replay, and migration before adding
  new state. Declare what is saved and what is runtime-only.
- Registry entries, backend handles, caches, scratch buffers, pools, derived
  indexes, and infrastructure objects should not be serialized unless there is a
  deliberate persistence contract.
- Public or constrained runtime layers must use their declared public helpers.
  They must not reach through to private backend, platform, or tooling objects.

## Defensive Coding

- Internal contracts should be direct and fail loudly when broken.
- Do not hide bugs with optional chaining, `typeof` function checks, null
  normalization, empty returns, lazy init guards, catch-and-return fallbacks, or
  missing-state blankets around objects that should exist by design.
- Defensive handling is valid at real boundaries: parsing, IO, network, browser
  or OS APIs, feature detection, user config, untrusted input, optional
  parameters, and explicitly optional interfaces.
- Throw for missing required configuration or impossible internal state. Do not
  return `null` as a bug blanket.

## Forbidden Shapes

- No legacy fallback paths unless the current supported runtime genuinely needs
  them.
- No descriptor-patterns.
- No facade, host, provider, service, adapter, manager, broker, or registry
  layers when they hide the real owner instead of expressing a domain mechanism.
- No wrapper-only functions. Single-call forwarding methods, private one-line
  delegators, and helpers that only bind constants into another helper are bad
  code unless they express a real domain contract outside a hot runtime path.
- No enterprise-style scaffolding in runtime code: object envelopes for a few
  scalars, generic option bags, callback plumbing layers, telemetry/readiness
  wrappers, broad "manager" APIs, defensive "just in case" branches,
  null-normalization, catch-and-continue fallbacks, or exception-shaped control
  flow are blockers unless a concrete boundary owner requires them.
- No analyzer suppressions, name/path exemptions, broad skips, or tags that make
  bad code look accepted.
- No local aliases of shared constants merely to shorten access. Read constants
  directly from their source table/module unless a local name captures a real
  concept, narrows a type, or avoids repeated expensive work.

## Async And Readiness

- Use established project primitives for real cross-owner async coordination,
  asset readiness, barriers, queues, or schedulers. A task gate is valid only
  when readiness is an externally observed owner contract, not as telemetry
  around local runtime/device work.
- Do not invent ad-hoc booleans, pending arrays, custom promise gates, task
  gates, or lazy initialization flags when direct lifecycle state can own the
  same fact.
- Initialization methods should normally be called exactly once by construction.

## Utilities

- Search common/shared utility areas before writing a new helper.
- Use existing helpers for operations like clamping, bounds checks, path
  handling, string normalization, geometry, scratch storage, and pooling.
- Add a helper only when it removes meaningful duplication, names a real concept,
  or centralizes a contract that already exists in multiple places.

## Performance

- Treat low-end hardware as a real target.
- Avoid fresh arrays, objects, closures, strings, normalized values, and parser
  work in hot paths. Use scratch buffers, typed arrays, object pools, cached
  parse results, precomputed lookup tables, or retained state when appropriate.
- In emulator, runtime, render, audio, input, scheduler, memory, and device code,
  allocation and GC churn are forbidden by default. New allocation is acceptable
  only at a real construction, IO/decoder/parser boundary, explicit async
  boundary, or ownership transfer, and the diff must make that reason obvious.
- Do not add enterprise-style generic state carriers, option objects, promise
  chains, task gates, maps, strings, or exception wrappers when direct device or
  runtime state can own the same fact.
- Do not add call-depth for the sake of neatness. Inline tiny pass-through
  operations in emulator hot paths, even when that duplicates a simple range
  test or device call.
- Keep string identifiers in constrained or hot runtime surfaces short and local.
- Precompute expensive parsing, matching, normalization, and dispatch work when
  the same result is used repeatedly.

## Modules And Tooling

- Use the project's normal module system in production code. Keep dynamic module
  loading and CommonJS-style loading to scripts, tools, or established boundary
  files where the project already permits it.
- Use the local package manager and project-local tools. Do not rely on global
  tool installs when a local dependency or script exists.
- For Node-based projects, prefer current LTS/current runtime behavior and local
  TypeScript/tooling. Verify the actual project requirement before changing the
  runtime contract.

## Lua Runtime And User-Layer Code

- Apply `lua-quality-rules.md` in addition to this file.
- Public/user-layer scripts must use the public helpers and declarative
  definitions exposed by their runtime layer. Do not require private runtime
  modules or call private backend objects when a public helper exists.
- Runtime Lua modules should own construction, compilation, scheduling, and hot
  path reuse. User-layer Lua should pass plain definitions, facts, and intent.

## C++ Runtime And Native Code

- Apply `cpp-quality-rules.md` in addition to this file.
- Native code should express ownership through direct object ownership, RAII,
  references, spans/views, scratch storage, and explicit protocol boundaries.
- Do not add pointer/null/optional fallbacks around required internal state.
- Keep headers and includes aligned with ownership. A lower layer must not
  include upward feature layers to reuse a convenience type.

## Validation

- Select checks that match the touched area: formatter, typecheck, unit test,
  analyzer, integration test, platform build, or runtime smoke test.
- Run the smallest useful checks while iterating, then the broader relevant
  entrypoint before finishing.
- Always run `git diff --check`.
