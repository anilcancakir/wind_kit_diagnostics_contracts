# Testing

Test a debug tool that reads wind_kit state without wind_kit: register a fake resolver and reset the registry between tests.

- [Debug Resolver](#debug-resolver)
- [Perf Resolver](#perf-resolver)
- [Isolation](#isolation)
- [Related Documentation](#related-documentation)

<a name="debug-resolver"></a>
## Debug Resolver

`registerForTesting` installs a fake into the debug slot. It is separate from `register` so a test never reads as production wiring.

```dart
import 'package:flutter/widgets.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:wind_kit_diagnostics_contracts/wind_kit_diagnostics_contracts.dart';

class _FakeResolver implements WindDebugResolver {
  @override
  Map<String, Object?> resolve(Element element) {
    return const <String, Object?>{
      'className': 'flex p-4',
      'breakpoint': 'lg',
      'brightness': 'light',
      'platform': 'web',
      'states': <String>['hover'],
    };
  }
}

void main() {
  setUp(WindDebugRegistry.resetForTesting);

  testWidgets('the tool emits the wind block', (WidgetTester tester) async {
    WindDebugRegistry.registerForTesting(_FakeResolver());
    await tester.pumpWidget(const SizedBox());

    final Element element = tester.element(find.byType(SizedBox));
    expect(WindDebugRegistry.current!.resolve(element)['className'], 'flex p-4');
  });
}
```

<a name="perf-resolver"></a>
## Perf Resolver

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:wind_kit_diagnostics_contracts/wind_kit_diagnostics_contracts.dart';

class _FakePerf implements WindPerfResolver {
  @override
  Map<String, Object?> stats() => const <String, Object?>{
        'cacheHits': 3,
        'widgetBuilds': <String, int>{'WDiv': 2},
      };
}

void main() {
  tearDown(WindDebugRegistry.resetForTesting);

  test('the tool reads the counters', () {
    WindDebugRegistry.registerPerfForTesting(_FakePerf());

    expect(WindDebugRegistry.currentPerf!.stats()['cacheHits'], 3);
    expect(WindDebugRegistry.current, isNull); // The debug slot is untouched.
  });
}
```

<a name="isolation"></a>
## Isolation

| Call | Effect |
|:-----|:-------|
| `resetForTesting()` | Clears both slots. Put it in `setUp` or `tearDown`; the registry is process-global. |
| `registerForTesting(fake)` | Fills the debug slot only. |
| `registerPerfForTesting(fake)` | Fills the perf slot only. |

A fake does not have to return every key. Return the ones your tool reads, and add a case for an empty map to cover the nothing-to-report path.

<a name="related-documentation"></a>
## Related Documentation

- [WindDebugRegistry](../basics/registry.md): the slots the fakes fill.
- [WindDebugResolver](../basics/debug-resolver.md): the keys a fake can return.
- [WindPerfResolver](../basics/perf-resolver.md): the counter keys.
