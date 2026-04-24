# Lua Quality Rules

These rules apply to Lua runtime modules and Lua user-layer scripts.

## Scope And Module Shape

- Prefer `local name<const> = function(...) ... end` for local functions when
  the Lua dialect supports `<const>`.
- Mark locals as `<const>` when they are never reassigned.
- Do not shadow module aliases imported with `require(...)`.
- `require(...)` paths must use module paths, not file suffixes.
- Build exported module tables directly at the destination. Do not stage a local
  table or call result only to assign it to a module field immediately afterward.
- Do not duplicate file-scope global-constant locals across files. Define the
  value once and reuse the owner.
- Keep public/user-layer code in its declared public surface. Do not reach into
  private runtime modules for convenience.

## Truthiness And Initialization

- Avoid `or nil`, empty-string fallbacks, boolean literal comparisons, and
  missing-value normalization that only makes bugs quieter.
- Required state should be created by construction and used directly.
- Lazy `ensure...` functions are debt unless the lifecycle is genuinely
  re-entrant or externally driven.
- Use optional checks only for truly optional values or boundary input.

## Numeric And Utility Use

- Bound values once at their owner or input boundary.
- Repeated clamps, floors, coordinate conversions, and size checks usually mean
  the code is missing a named concept or utility.
- Use existing numeric and utility helpers before writing local variants.

## Events

- Treat events as announcements, not command buses.
- Event names should be short, local, and stable.
- Payloads should contain the facts subscribers need, not private runtime
  handles.
- Avoid request/reply protocols, relay flags, pending maps, and event fanout
  layers when direct ownership is available.
- Persistent subscriptions belong to long-lived listeners. Short-lived flows
  should use explicit lifecycle ownership.

## State Machines

- State ownership belongs to the state-machine owner.
- Avoid cross-state boolean flags that duplicate transition state.
- Async waits should use the project event/wait primitive rather than polling
  ticks, frame counters, or local pending arrays.
- Dispatch through direct handler references or explicit tables, not generic
  stringly middleware.

## Timelines And Declarative Definitions

- Declarative definitions should describe intent and facts, not prebuilt
  internal runtime objects.
- Keep identifiers short where they are frequently compared or stored.
- Avoid repeated namespace prefixes in tags, events, effect IDs, timeline IDs,
  metrics, and protocol keys.

## Object Lifecycle And Registries

- Runtime registries, caches, scratch buffers, pools, indexes, and backend
  handles are runtime infrastructure and should not be serialized by default.
- Lifecycle hooks should update all owned state consistently. Do not patch one
  side table when the owner exposes a method that updates the full state.

## Systems And Hot Paths

- Hot paths must not allocate fresh tables, closures, strings, parser results, or
  temporary arrays per frame/tick/sample/input.
- Precompute parser, lookup, dispatch, and normalized string work.
- Reuse scratch tables or retained buffers where the project has that pattern.

## Rendering Or Low-Level Runtime Boundaries

- Low-level draw, audio, scheduling, and input code should keep data flat and
  direct.
- User-layer code should pass definitions and intent. Runtime code owns
  compilation, scheduling, and retained buffers.

## Analyzer Expectations

- Lua rules should use parser and scope data: table fields, method calls,
  assignment flow, local binding scopes, module aliases, string literals, and
  function bodies matter.
- Keep runtime-layer and user-layer profiles explicit.
- Suppression comments must be local, rule-specific, and reasoned.
- Analyzer fixes must improve ownership or rule precision.
