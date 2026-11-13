# WindDebugResolver

Read the resolved wind_kit state of any `Element`: class name, breakpoint, brightness, platform, states and colours.

- [Basic Usage](#basic-usage)
- [Quick Reference](#quick-reference)
- [Implementing a Resolver](#implementing-a-resolver)
- [Frozen Contract](#frozen-contract)
- [Gotchas](#gotchas)
- [Related Documentation](#related-documentation)

<a name="basic-usage"></a>
## Basic Usage

A debug tool walks the element tree and asks the registered resolver about each element. An element with nothing to report (not a W-widget, or one without a `className`) resolves to an empty map, so the walk is safe over any tree.

```dart
import 'package:flutter/widgets.dart';
import 'package:wind_kit_diagnostics_contracts/wind_kit_diagnostics_contracts.dart';

void emitWindBlock(StringBuffer buffer, Element element) {
  final WindDebugResolver? resolver = WindDebugRegistry.current;
  if (resolver == null) return; // wind_kit is not in this app, or this is a release build.

  final Map<String, Object?> data = resolver.resolve(element);
  if (data.isEmpty) return; // Nothing to report for this element.

  buffer.writeln('wind:');
  data.forEach((String key, Object? value) {
    buffer.writeln('  $key: $value');
  });
}
```

<a name="quick-reference"></a>
## Quick Reference

`resolve(Element element)` returns `Map<String, Object?>`. The key set is the v1 contract:

| Key | Type | Meaning |
|:----|:-----|:--------|
| `className` | `String` | The widget's `className`. Present for a W-widget with a non-empty one. |
| `breakpoint` | `String` | The active breakpoint, such as `sm`, `md` or `lg`. |
| `brightness` | `String` | `light` or `dark`. |
| `platform` | `String` | `web`, `ios`, `android`, `macos` and so on. |
| `states` | `List<String>` | The active interaction states, such as `hover` or `focus`. |
| `bgColor` | `String` | Background colour as hex, such as `#3B82F6`. Present only when resolved. |
| `textColor` | `String` | Text colour as hex. Present only when resolved. |

<a name="implementing-a-resolver"></a>
## Implementing a Resolver

`wind_kit` ships the production resolver. Implement the contract yourself only for a test fake, or in an app that styles with another library instead of wind_kit. There is one slot per app, so a second implementation next to wind_kit's would replace it.

```dart
import 'package:flutter/widgets.dart';
import 'package:wind_kit_diagnostics_contracts/wind_kit_diagnostics_contracts.dart';

class MyStyleResolver implements WindDebugResolver {
  const MyStyleResolver();

  @override
  Map<String, Object?> resolve(Element element) {
    final Widget widget = element.widget;
    if (widget is! MyStyledBox) return const <String, Object?>{};

    return <String, Object?>{
      'className': widget.className,
      'brightness': 'light',
    };
  }
}

class MyStyledBox extends StatelessWidget {
  const MyStyledBox({super.key, required this.className});

  final String className;

  @override
  Widget build(BuildContext context) => const SizedBox.shrink();
}
```

Return `const {}` for every element that is not yours. Consumers rely on that to walk arbitrary trees.

<a name="frozen-contract"></a>
## Frozen Contract

The key set is frozen for the 1.x line. A new key is a minor bump, and renaming or removing a key is a major one, so a reader ignores keys it does not know. See [Versioning](../guides/versioning.md).

<a name="gotchas"></a>
## Gotchas

| Mistake | Fix |
|:--------|:----|
| Asserting that the map has exactly the keys above | Read the keys you need and ignore the rest; minor releases add keys. |
| Treating a missing `bgColor` as an error | It is present only when the colour resolved. |
| Printing `states` raw in a one-line snapshot | It is a `List<String>`; join it (`states.join(',')`). |
| Returning `null` from `resolve` for a foreign element | The signature is non-nullable: return `const {}`. |

<a name="related-documentation"></a>
## Related Documentation

- [WindDebugRegistry](registry.md): where the resolver is installed and looked up.
- [WindPerfResolver](perf-resolver.md): the contract for process-wide counters.
- [Testing](../guides/testing.md): registering a fake in a widget test.
