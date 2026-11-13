# WindDebugRegistry

The process-global registry that connects the side that implements the contracts (`wind_kit`) to the side that reads them (a debug tool).

- [Basic Usage](#basic-usage)
- [Quick Reference](#quick-reference)
- [Two Slots](#two-slots)
- [Who Installs](#who-installs)
- [Gotchas](#gotchas)
- [Related Documentation](#related-documentation)

<a name="basic-usage"></a>
## Basic Usage

One side registers a resolver, the other looks it up on every call.

```dart
import 'package:flutter/widgets.dart';
import 'package:wind_kit_diagnostics_contracts/wind_kit_diagnostics_contracts.dart';

// Implementing side: install once at boot.
void install(WindDebugResolver resolver) {
  WindDebugRegistry.register(resolver);
}

// Reading side: look the resolver up per call.
Map<String, Object?> windState(Element element) {
  final WindDebugResolver? resolver = WindDebugRegistry.current;
  if (resolver == null) return const <String, Object?>{};

  return resolver.resolve(element);
}
```

<a name="quick-reference"></a>
## Quick Reference

| Member | Type | Meaning |
|:-------|:-----|:--------|
| `current` | `static WindDebugResolver?` | The registered debug resolver, or `null` when none was installed. |
| `currentPerf` | `static WindPerfResolver?` | The registered perf resolver, or `null`. |
| `register(resolver)` | `static void` | Installs the debug resolver. Idempotent; the most recent call wins. |
| `registerPerf(resolver)` | `static void` | Installs the perf resolver, in its own slot. Same rules. |
| `resetForTesting()` | `static void` | Test-only. Clears both slots. |
| `registerForTesting(resolver)` | `static void` | Test-only. Puts a fake in the debug slot. |
| `registerPerfForTesting(resolver)` | `static void` | Test-only. Puts a fake in the perf slot. |

The class cannot be instantiated; every member is static.

<a name="two-slots"></a>
## Two Slots

The debug slot and the perf slot are independent. Registering one never touches the other, and `resetForTesting()` is the only call that clears both. They are separate contracts because they answer different questions: [WindDebugResolver](debug-resolver.md) resolves one `Element`, [WindPerfResolver](perf-resolver.md) reports counters that belong to no element.

<a name="who-installs"></a>
## Who Installs

The host app installs both through wind_kit at boot, behind `!kReleaseMode`, so a debug or profile build carries them and a release build tree-shakes the call. wind_kit never makes these calls itself:

```dart
import 'package:flutter/foundation.dart';
import 'package:wind_kit/wind_kit.dart';

void installWindDiagnostics() {
  if (kReleaseMode) return;

  Wind.installDebugResolver();
  Wind.installPerfResolver();
}
```

Each method is idempotent. In a release build both slots stay `null`, and so they do in a wind_kit app whose `main` never makes the calls.

<a name="gotchas"></a>
## Gotchas

| Mistake | Fix |
|:--------|:----|
| Caching `WindDebugRegistry.current` in a field at startup | Read it on every use. The host may install after your tool starts, or never. |
| Reading `null` as "this is not a wind_kit widget" | `null` means no resolver is registered: wind_kit is absent, this is a release build, or the host never made the install calls. An element with nothing to report resolves to an empty map. |
| Expecting a notification when a resolver is installed | There is none: the registry is two nullable getters. Poll on use. |
| Calling the `*ForTesting` methods from `lib/` | They are `@visibleForTesting`. Production code installs through `register` and `registerPerf`. |
| Registering a second resolver next to wind_kit's | The most recent call wins, so yours would replace wind_kit's. |

<a name="related-documentation"></a>
## Related Documentation

- [WindDebugResolver](debug-resolver.md): the per-element contract.
- [WindPerfResolver](perf-resolver.md): the counters contract.
- [Testing](../guides/testing.md): fakes and `resetForTesting()`.
