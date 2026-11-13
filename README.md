<p align="center">
  <img src="https://raw.githubusercontent.com/anilcancakir/wind_kit_diagnostics_contracts/master/.github/wind-kit-logo.svg" width="120" alt="wind_kit_diagnostics_contracts logo" />
</p>

<h1 align="center">wind_kit_diagnostics_contracts</h1>

<p align="center">
  <strong>Read wind_kit widget state without depending on wind_kit.</strong><br/>
  Pure abstract contracts and a process-global registry, so debug tools and AI agents see className, breakpoint, brightness and states while wind_kit and the tool stay strangers. Frozen v1 contract.
</p>

<p align="center">
  <a href="https://pub.dev/packages/wind_kit_diagnostics_contracts"><img src="https://img.shields.io/pub/v/wind_kit_diagnostics_contracts.svg" alt="pub package"></a>
  <a href="https://github.com/anilcancakir/wind_kit_diagnostics_contracts/actions/workflows/ci.yml"><img src="https://img.shields.io/github/actions/workflow/status/anilcancakir/wind_kit_diagnostics_contracts/ci.yml?branch=master&label=CI" alt="CI"></a>
  <a href="https://codecov.io/gh/anilcancakir/wind_kit_diagnostics_contracts"><img src="https://codecov.io/gh/anilcancakir/wind_kit_diagnostics_contracts/graph/badge.svg" alt="codecov"></a>
  <a href="https://opensource.org/licenses/MIT"><img src="https://img.shields.io/badge/License-MIT-blue.svg" alt="License: MIT"></a>
  <a href="https://pub.dev/packages/wind_kit_diagnostics_contracts/score"><img src="https://img.shields.io/pub/points/wind_kit_diagnostics_contracts" alt="pub points"></a>
</p>

<p align="center">
  <a href="https://anilcancakir.com/wind_kit_diagnostics_contracts">Documentation</a> ·
  <a href="https://pub.dev/packages/wind_kit_diagnostics_contracts">pub.dev</a> ·
  <a href="https://github.com/anilcancakir/wind_kit_diagnostics_contracts/issues">Issues</a>
</p>

## At a glance

| Fact | Value |
|---|---|
| Package | [`wind_kit_diagnostics_contracts`](https://pub.dev/packages/wind_kit_diagnostics_contracts) |
| Install | `flutter pub add wind_kit_diagnostics_contracts` |
| Import | `package:wind_kit_diagnostics_contracts/wind_kit_diagnostics_contracts.dart` |
| Entry points | `WindDebugResolver` (per-element widget state), `WindPerfResolver` (process-wide counters), `WindDebugRegistry` (the two slots) |
| Platforms | All: plain Dart over Flutter's `Element`, no UI, no native code |
| Requires | Dart >=3.4, Flutter >=3.27 |
| Dependencies | `flutter` and `meta` only |
| License | MIT |

## Install

```bash
flutter pub add wind_kit_diagnostics_contracts
```

Most apps never add it by hand: `wind_kit` depends on it, so any app that uses wind_kit already has it. Add it when you author a debug tool that reads wind_kit state, or a styling library that exposes its own.

## Why it exists

[wind_kit](https://pub.dev/packages/wind_kit) knows each widget's resolved state (`className`, `breakpoint`, `brightness`, `platform`, `states`, colours). Debug tools such as [tapir](https://pub.dev/packages/tapir) put that state in their snapshots so an LLM agent can reason about the rendered tree. Handing it over through wind_kit's own API would make every tool depend on wind_kit's whole rendering surface; the other way round would make wind_kit depend on every tool.

This package breaks the loop. Both sides depend on the contract here, and neither imports the other. It follows Flutter's `*_platform_interface` convention.

```
          wind_kit_diagnostics_contracts
                      |
        +-------------+-------------+
        |                           |
    wind_kit                      tapir
 (registers a resolver)   (reads it at snapshot time)
```

## WindDebugResolver

One method, `resolve(Element)`, answers per-element widget state. A non-wind_kit element resolves to an empty map, so a tool can walk any tree.

```dart
import 'package:flutter/widgets.dart';
import 'package:wind_kit_diagnostics_contracts/wind_kit_diagnostics_contracts.dart';

void emitWindBlock(StringBuffer buffer, Element element) {
  final WindDebugResolver? resolver = WindDebugRegistry.current;
  if (resolver == null) return; // wind_kit is not in this app, or this is a release build.

  final Map<String, Object?> data = resolver.resolve(element);
  if (data.isEmpty) return; // Not a wind_kit widget.

  buffer.writeln('wind:');
  data.forEach((String key, Object? value) => buffer.writeln('  $key: $value'));
}
```

| Key | Type | Present |
|-----|------|---------|
| `className` | `String` | For a W-widget with a non-empty `className` |
| `breakpoint` | `String` | `sm`, `md`, `lg`, ... |
| `brightness` | `String` | `light` or `dark` |
| `platform` | `String` | `web`, `ios`, `android`, `macos`, ... |
| `states` | `List<String>` | `hover`, `focus`, ... |
| `bgColor`, `textColor` | `String` (hex) | Only when resolved |

## WindPerfResolver

A second, independent contract for numbers that belong to no single element: cache hits, build counts, inherited-widget reads. It has its own slot, and registering one contract never touches the other.

```dart
import 'package:wind_kit_diagnostics_contracts/wind_kit_diagnostics_contracts.dart';

Map<String, Object?>? readStats() {
  final WindPerfResolver? perf = WindDebugRegistry.currentPerf;
  if (perf == null) return null; // No resolver was ever registered.

  return perf.stats();
}
```

`stats()` returns `cacheHits`, `cacheMisses`, `cacheBypasses`, `cacheSize`, `wDivBuilds` and `wTextBuilds` as `int`, plus `widgetBuilds`, `wrapperEmissions` and `inheritedReads` as `Map<String, int>`. Keep the `null`: it tells "never registered" apart from "the counters really are zero".

## Registering

wind_kit installs both resolvers at app boot behind `!kReleaseMode`, so a release build tree-shakes them:

```dart
import 'package:flutter/foundation.dart';
import 'package:wind_kit/wind_kit.dart';

void main() {
  if (!kReleaseMode) {
    Wind.installDebugResolver();
    Wind.installPerfResolver();
  }
}
```

## Testing

`WindDebugRegistry` ships three `@visibleForTesting` seams: `resetForTesting()` clears both slots, `registerForTesting` and `registerPerfForTesting` fill one each. See the [testing guide](https://anilcancakir.com/wind_kit_diagnostics_contracts/docs/guides/testing).

## For AI coding agents

Rules for an agent writing code against this package. Each one is a mistake the maintainers already made once.

| Do | Because |
|----|---------|
| Import only `package:wind_kit_diagnostics_contracts/wind_kit_diagnostics_contracts.dart` | It is the only library; everything is public through it. |
| Read `WindDebugRegistry.current` and `currentPerf` on every use and handle `null` | The host may install after your tool starts, or never (release build, no wind_kit). `null` means no resolver; an empty map means a non-wind_kit element. |
| Read the keys you need and ignore the rest | A minor release adds keys. Renaming or removing one is a major bump. |
| Take per-type build counts from `widgetBuilds`, not `wDivBuilds` or `wTextBuilds` | The two older keys duplicate `widgetBuilds['WDiv']` and `['WText']` and stay only for the 1.x line. |
| Register through `register` and `registerPerf`, and call `*ForTesting` only from `test/` | The testing seams are `@visibleForTesting`; a fake installed through `register` reads as production wiring. |
| Call `WindDebugRegistry.resetForTesting()` in `setUp` or `tearDown` | The registry is process-global and survives between tests. |
| Never add a method to either contract | It breaks every implementer, test fakes included, so it is a major bump. Add a key, or a new contract. |

## Dependencies

`flutter` (for `Element`) and `meta` (for `@visibleForTesting`). That is the whole surface, on purpose: a dependency added here is a dependency forced on wind_kit and every tool.

## Links

- [Documentation](https://anilcancakir.com/wind_kit_diagnostics_contracts/docs/getting-started/installation): installation, the registry, both resolvers, testing and versioning
- [API reference](https://pub.dev/documentation/wind_kit_diagnostics_contracts/latest/)
- [llms.txt](llms.txt) for AI coding agents
- [Changelog](CHANGELOG.md)
- [Report an issue](https://github.com/anilcancakir/wind_kit_diagnostics_contracts/issues)
- [Security policy](SECURITY.md)

## License

MIT, see [LICENSE](LICENSE). Built by [Anılcan Çakır](https://anilcancakir.com).
