# wind_kit_diagnostics_contracts

Pure abstract contracts that let debug tools read wind_kit widget state (`WindDebugResolver`) and performance counters (`WindPerfResolver`) through a process-global registry (`WindDebugRegistry`), without depending on wind_kit. wind_kit registers the resolvers; tapir reads them. Neither imports the other.

**Version:** 1.3.0 · **Dart:** >=3.4.0 · **Flutter:** >=3.27.0

## Commands

| Command | Description |
|---------|-------------|
| `flutter test --coverage` | Run all tests with coverage |
| `flutter analyze --no-fatal-infos` | Static analysis |
| `dart format .` | Format all code |
| `flutter pub publish --dry-run` | Pre-publish check |
| `python3 tool/check-docs.py` | Docs structure, links, fragments and ToC anchors; run it after any `doc/` change |

## Architecture

```
lib/
└── wind_kit_diagnostics_contracts.dart   # The whole public API: the two contracts and the registry
test/
└── wind_kit_diagnostics_contracts_test.dart
example/example.md        # A reader of the registry (pana accepts it as the example)
doc/                      # User docs, published by the docs site: getting-started/, basics/, guides/
tool/check-docs.py        # Offline docs gate, run by docs-link-check.yml
llms.txt                  # Agent-facing summary and rules
```

There is no agent skill: the contracts are read through the wind_kit and tapir skills.

## Post-Change Checklist

After ANY source code change, sync **before committing**. A change is not done while one of these is stale:

1. **`CHANGELOG.md`**: Add entry under `[Unreleased]` section
2. **`README.md`, `llms.txt` and the `doc/` pages**: Update if the API, a key or a rule changes
3. **`CLAUDE.md`**: Update the architecture tree when files move

A change to a returned key set is a coordinated bump across wind_kit and tapir, which both pin `^1.x`.

## Development Flow (TDD)

Every feature, fix, or refactor must go through the red-green-refactor cycle:

1. **Red**: Write a failing test that describes the expected behavior
2. **Green**: Write the minimum code to make the test pass
3. **Refactor**: Clean up while keeping tests green

**Rules:**
- No production code without a failing test first
- Run `flutter test` after every change: all tests must stay green
- Run `flutter analyze --no-fatal-infos` after every change: zero warnings, zero errors
- Run `dart format .` before committing: zero formatting issues

## Testing

- Tests live in `test/wind_kit_diagnostics_contracts_test.dart` and use fakes that implement the two contracts; no mocking library
- Call `WindDebugRegistry.resetForTesting()` in `setUp`: the registry is process-global
- Import the barrel, `package:wind_kit_diagnostics_contracts/wind_kit_diagnostics_contracts.dart`

## Key Gotchas

| Mistake | Fix |
|---------|-----|
| Adding a method to `WindDebugResolver` or `WindPerfResolver` | A major bump: it breaks every implementer, test fakes included. Add a key to the returned map, or a new contract (the perf contract exists for this reason) |
| Renaming or removing a key in a returned map | A major bump. `wDivBuilds` and `wTextBuilds` stay for the 1.x line even though `widgetBuilds` carries the same counts |
| Registering the perf resolver through `register` | That fills the debug slot; use `registerPerf` |
| Adding a dependency | It is forced on wind_kit and every tool. `flutter` and `meta` are the whole surface |
| Importing `wind_kit` or `tapir` | Never: this package is the neutral bridge |
| Removing `// coverage:ignore-line` from `WindDebugRegistry._()` | The private constructor is unreachable from a test; the pragma keeps codecov at 100% |
| Removing `analyzer.exclude` from `analysis_options.yaml` | The Flutter migrator rewrites it on every `pub get`, and an uncommitted rewrite fails `flutter pub publish --dry-run` with exit 65 |
| Relative paths in `pubspec_overrides.yaml` | Use ABSOLUTE ones; the file is never committed |

## Release

1. Open a `release:` PR that bumps `pubspec.yaml` and the `Version:` line above, and promotes `[Unreleased]` in `CHANGELOG.md`.
2. After it merges, tag the merge commit with the bare version (`git tag 1.3.1 && git push origin 1.3.1`, no `v`). `publish.yml` runs from the tag; the pub.dev Admin tab holds repository `anilcancakir/wind_kit_diagnostics_contracts`, tag pattern `{{version}}` and push events only.
3. A failed `validate` is fixed forward: delete the tag (`git push --delete origin <tag>` and `git tag -d <tag>`), fix it on a PR and tag the new merge commit, because a re-run builds the same commit. If only `github-release` fails after pub.dev accepted the version, use "Re-run failed jobs" (never "Re-run all jobs", where `publish` is rejected as a duplicate) or run `gh release create <tag> --title <tag> --latest --verify-tag --notes-file <file>`.

The first publish of this name (1.3.0) is manual, since tag-triggered OIDC cannot create a package: `dart pub publish --force` from a clean checkout of the merge commit, then the tag.

## CI

- `ci.yml`: push/PR → `flutter pub get` → `flutter analyze --no-fatal-infos` → `dart format --set-exit-if-changed` → `flutter test --coverage` → codecov upload
- `changelog.yml`: a PR that changes `lib/` must also change `CHANGELOG.md`; Dependabot, `release:` PRs and the `skip-changelog` label are exempt
- `docs-link-check.yml`: every PR runs `python3 tool/check-docs.py` and an offline lychee pass; Mondays an external-link pass
- `publish.yml`: tag on master → version stamps (tag, pubspec) → analyze, format, test, dry-run, pana 160/160, changelog section → OIDC publish → GitHub Release
