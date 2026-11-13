# Installation

Add `wind_kit_diagnostics_contracts` to a package that reads or serves wind_kit widget state, and know when you do not need it.

- [Install](#install)
- [Who Needs It](#who-needs-it)
- [Requirements](#requirements)
- [What You Do Not Need](#what-you-do-not-need)
- [Verify](#verify)
- [Related Documentation](#related-documentation)

<a name="install"></a>
## Install

```bash
flutter pub add wind_kit_diagnostics_contracts
```

The only import is `package:wind_kit_diagnostics_contracts/wind_kit_diagnostics_contracts.dart`. It exports `WindDebugResolver`, `WindPerfResolver` and `WindDebugRegistry`.

<a name="who-needs-it"></a>
## Who Needs It

| You are | Add it? |
|:--------|:--------|
| An app that uses `wind_kit` | No. `wind_kit` depends on it, so you already have it. |
| A debug tool that reads wind_kit state (`tapir`, an inspector, a DevTools extension) | Yes, as a direct dependency. Do not depend on `wind_kit` itself. |
| A styling library that exposes its own state to the same tools | Yes, and implement `WindDebugResolver`. |
| A test that needs a fake resolver | Yes, as a `dev_dependency`. |

<a name="requirements"></a>
## Requirements

| Requirement | Value |
|:------------|:------|
| Dart | `>=3.4.0` |
| Flutter | `>=3.27.0` |
| Dependencies | `flutter` and `meta` |
| Platforms | All: the package is plain Dart over Flutter's `Element` type, with no UI and no native code |

<a name="what-you-do-not-need"></a>
## What You Do Not Need

- No `wind_kit` dependency: that is the point of the package.
- No setup, config file, code generation or CLI command.
- No registration of your own. `wind_kit` installs its resolvers at app boot, and until it does both registry slots are `null`.

<a name="verify"></a>
## Verify

```dart
import 'package:flutter/foundation.dart';
import 'package:wind_kit_diagnostics_contracts/wind_kit_diagnostics_contracts.dart';

void main() {
  debugPrint('debug resolver: ${WindDebugRegistry.current}');
  debugPrint('perf resolver: ${WindDebugRegistry.currentPerf}');
}
```

Both lines print `null` until the host installs a resolver. In an app that calls `Wind.installDebugResolver()` and `Wind.installPerfResolver()` in a debug or profile build, they print the installed implementations.

<a name="related-documentation"></a>
## Related Documentation

- [WindDebugRegistry](../basics/registry.md): the two slots and who fills them.
- [WindDebugResolver](../basics/debug-resolver.md): per-element widget state.
- [WindPerfResolver](../basics/perf-resolver.md): process-wide counters.
- [Testing](../guides/testing.md): fake resolvers in tests.
