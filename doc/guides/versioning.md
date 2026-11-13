# Versioning

What counts as a breaking change in this package, so a reader and an implementer know what the 1.x line promises.

- [Rules](#rules)
- [For Readers](#for-readers)
- [For Implementers](#for-implementers)
- [Related Documentation](#related-documentation)

<a name="rules"></a>
## Rules

The package follows [Semantic Versioning 2.0.0](https://semver.org/spec/v2.0.0.html). The load-bearing contract is two key sets: the map `WindDebugResolver.resolve` returns and the map `WindPerfResolver.stats` returns.

| Change | Bump |
|:-------|:-----|
| A new key in a returned map | minor |
| A new registry slot or static member | minor |
| Renaming or removing a key | major |
| A new method on `WindDebugResolver` or `WindPerfResolver` | major |

A new method is breaking because it breaks every implementer, the test fakes included. That is why the perf counters arrived as a second contract in 1.1.0 and not as a second method on the first.

<a name="for-readers"></a>
## For Readers

Read the keys you need and ignore the rest. A map that grows a key in a minor release must not break your tool.

<a name="for-implementers"></a>
## For Implementers

Return every key you document, and return `const {}` from `resolve` for an element that is not yours. Pin the dependency with a caret (`^1.3.0`): a minor release adds keys and never removes one.

<a name="related-documentation"></a>
## Related Documentation

- [WindDebugResolver](../basics/debug-resolver.md): the debug key set.
- [WindPerfResolver](../basics/perf-resolver.md): the counter key set.
