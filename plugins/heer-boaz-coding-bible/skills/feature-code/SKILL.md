---
name: feature-code
description: "Use when implementing features, bug fixes, runtime behavior, user-facing workflows, editor or IDE behavior, media/render/audio/video work, platform support, APIs, Lua code, C++ code, or production code changes in any project. Add code only after identifying the real owner and the active architecture contract. If the change requires cleanup first, stop and use lean-code."
---

# Feature Code

Use this for normal implementation work. Find the owner, add the behavior there,
and do not make touched code worse.

## Before Editing

Read nearby code and the relevant shared references:

- `../../references/coding-architecture-rules.md` for general doctrine.
- `../../references/anti-patterns.md` when the touched area smells messy.
- `../../references/lua-quality-rules.md` when touching Lua runtime or Lua user-layer code.
- `../../references/cpp-quality-rules.md` when touching C++ runtime, platform, or native code.
- `../../references/reference-implementation-guide.md` before unfamiliar domain work.

Search with `rg` for existing owners, state, helpers, public APIs, tests, and
language/runtime counterparts. Prefer established project primitives over new
local patterns.

## Feature Gate

Feature code is allowed only when:

- the behavior has a concrete owner;
- the public/runtime contract remains explicit;
- internal invariants fail loudly instead of being hidden;
- hot paths stay allocation-conscious;
- serialization, persistence, or state replay implications are understood;
- validation can exercise the touched area.

Stop and switch to `lean-code` if the feature would require wrappers, lazy init,
defensive clutter, facade/host/provider/service layers, descriptor indirection,
hidden analyzer skips, compatibility fallbacks, or broad cleanup first.

When touching bad code, keep the feature slice small. Do not copy the mess and
do not present tagged or suppressed code as clean. If a local comment or quality
tag is unavoidable, make it say why the code is exceptional or still debt.

## Validation

Use the existing project build, test, analyzer, and validation entrypoints for
the touched area. Add narrower runs only when they increase confidence. Always
run `git diff --check` before finishing.
