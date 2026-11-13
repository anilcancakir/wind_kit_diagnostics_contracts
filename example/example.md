# wind_kit_diagnostics_contracts example

Read the wind_kit state of an element from a debug tool, without depending on wind_kit.

```dart
import 'package:flutter/widgets.dart';
import 'package:wind_kit_diagnostics_contracts/wind_kit_diagnostics_contracts.dart';

/// Writes a `wind:` block for [element], or nothing when wind_kit has not
/// registered a resolver or the element has nothing to report.
void emitWindBlock(StringBuffer buffer, Element element) {
  final WindDebugResolver? resolver = WindDebugRegistry.current;
  if (resolver == null) return;

  final Map<String, Object?> data = resolver.resolve(element);
  if (data.isEmpty) return;

  buffer.writeln('wind:');
  data.forEach((String key, Object? value) => buffer.writeln('  $key: $value'));
}
```

The host app installs the resolver at boot, so the registry is filled before a tool reads it:

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
