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
- No wrapper-only functions.
- No analyzer suppressions, name/path exemptions, broad skips, or tags that make
  bad code look accepted.
- No local aliases of shared constants merely to shorten access. Read constants
  directly from their source table/module unless a local name captures a real
  concept, narrows a type, or avoids repeated expensive work.

## Async And Readiness

- Use established project primitives for async coordination, asset readiness,
  task gates, barriers, queues, or schedulers.
- Do not invent ad-hoc booleans, pending arrays, custom promise gates, or lazy
  initialization flags when the project already has a real coordination model.
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
