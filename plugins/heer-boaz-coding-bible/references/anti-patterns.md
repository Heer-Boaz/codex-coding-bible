# Anti-Patterns

Use this as a quick smell list before and after editing.

## Defensive Clutter

Avoid internal code shaped like:

```ts
if (!platform) return null;
if (!host || typeof host.getCapability !== 'function') return null;
try { return provider.getMetrics(); } catch { return null; }
```

Prefer a strict contract:

```ts
return runtime.platform.metrics.getMetrics();
```

Valid exceptions are external input, parsing, IO, network, optional user-provided
configuration, feature detection, and explicitly optional APIs.

## Public Shortcut Leakage

Avoid exposing platform conveniences, backend handles, test shortcuts, or private
implementation details as public runtime behavior. If downstream code can
observe or depend on a feature, route it through the declared public contract.

## Ad-Hoc Async Readiness

Avoid one-off booleans, local pending arrays, custom promise gates, and repeated
"ensure initialized" guards. Use the project's established readiness primitive
or make the lifecycle direct enough that no guard is needed.

## Nullish Normalization

Avoid:

```ts
const value = maybeValue ?? null;
return value;
```

Prefer preserving the actual contract:

```ts
return maybeValue;
```

Only convert to `null` at a public boundary that explicitly requires `null`.

## Useless Local Constants

Avoid locals that name one obvious use or only satisfy a rule:

```ts
const current = record.current;
return current;
```

Prefer the direct expression:

```ts
return record.current;
```

Create a local only when it names a real concept, avoids repeated work, narrows
a type, or improves performance.

## Optional-Chain Bug Hiding

Avoid:

```ts
this.device?.tick?.();
```

Prefer:

```ts
this.device.tick();
```

Use optional chaining only when the property or method is truly optional in the
type and domain contract.

## Catch Fallbacks

Avoid:

```ts
try {
    return buildInternalState();
} catch {
    return null;
}
```

Prefer letting internal failures surface. If the catch is a true boundary,
handle it explicitly and explain why.

## Closed-Kind Dispatch

Prefer `switch` or table-driven dispatch when the set is explicit. Long chains
of repeated kind comparisons are usually a missing dispatch concept.

## Repeated Semantic Work

Repeated `toLowerCase`, `trim`, `split`, `join`, `Math.min`, `Math.max`,
`floor`, clamp calls, bounds checks, caret normalization, line splitting,
keyword lookup, or query normalization usually means a missing concept.

## Analyzer Evasion

Never rewrite a violation into a larger equivalent violation. If the analyzer
rule is too strict or too loose, improve the analyzer first.

## Serialization Leaks

Avoid putting registries, backend handles, caches, scratch buffers, pools,
derived indexes, or runtime infrastructure into save data. New feature state
should declare what is saved and what is intentionally excluded.
