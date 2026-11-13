# WindPerfResolver

Read process-wide wind_kit performance counters (cache hits, build counts, inherited-widget reads) that belong to no single element.

- [Basic Usage](#basic-usage)
- [Quick Reference](#quick-reference)
- [Null Versus Zero](#null-versus-zero)
- [Frozen Contract](#frozen-contract)
- [Gotchas](#gotchas)
- [Related Documentation](#related-documentation)

<a name="basic-usage"></a>
## Basic Usage

```dart
import 'package:wind_kit_diagnostics_contracts/wind_kit_diagnostics_contracts.dart';

Map<String, int> widgetBuilds() {
  final WindPerfResolver? perf = WindDebugRegistry.currentPerf;
  if (perf == null) return const <String, int>{}; // No resolver was ever registered.

  final Map<String, Object?> stats = perf.stats();

  return (stats['widgetBuilds'] as Map<String, int>?) ?? const <String, int>{};
}
```

<a name="quick-reference"></a>
## Quick Reference

`stats()` returns `Map<String, Object?>`:

| Key | Type | Meaning |
|:----|:-----|:--------|
| `cacheHits` | `int` | Style cache hits. |
| `cacheMisses` | `int` | Style cache misses. |
| `cacheBypasses` | `int` | Lookups that skipped the cache. |
| `cacheSize` | `int` | Entries in the cache. |
| `wDivBuilds` | `int` | `WDiv` builds. Equals `widgetBuilds['WDiv']`. |
| `wTextBuilds` | `int` | `WText` builds. Equals `widgetBuilds['WText']`. |
| `widgetBuilds` | `Map<String, int>` | W-widget type name to build count. |
| `wrapperEmissions` | `Map<String, int>` | Flutter wrapper type that wind_kit emitted, to count. |
| `inheritedReads` | `Map<String, int>` | Counted reads of `mediaQuerySize`, `mediaQueryBrightness`, `windTheme` and `defaultTextStyle`. |

`wDivBuilds` and `wTextBuilds` stay for the 1.x line because removing a key needs a major bump. A new reader takes per-type counts from `widgetBuilds`.

<a name="null-versus-zero"></a>
## Null Versus Zero

Pass the `null` through instead of flattening it to zeros. `currentPerf == null` means no resolver was ever registered; a registered resolver that reports zeros means the counters really are zero. wind_kit counts only while its own `WindPerfCounters.enabled` flag is on, so installing the resolver costs nothing on its own.

<a name="frozen-contract"></a>
## Frozen Contract

Same rule as [WindDebugResolver](debug-resolver.md): new keys are a minor bump, a rename or removal is a major one. Adding a method to this contract is also a major bump, which is why the counters live in a second contract instead of a second method on the first. See [Versioning](../guides/versioning.md).

<a name="gotchas"></a>
## Gotchas

| Mistake | Fix |
|:--------|:----|
| Reading `wDivBuilds` for a new feature | Read `widgetBuilds['WDiv']`. |
| Reading `stats()` for per-element data | Counters have no element; use [WindDebugResolver](debug-resolver.md). |
| Registering the perf resolver with `register` | Use `registerPerf`; `register` fills the debug slot. |
| Assuming zeros mean nothing happened | The counters may be switched off; check wind_kit's flag first. |

<a name="related-documentation"></a>
## Related Documentation

- [WindDebugRegistry](registry.md): the perf slot and `currentPerf`.
- [Versioning](../guides/versioning.md): what counts as a breaking change.
- [Testing](../guides/testing.md): `registerPerfForTesting`.
