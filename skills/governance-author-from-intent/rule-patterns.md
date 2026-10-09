# Rule patterns and pitfalls

Reference for writing Validation Profile 1.0 rules once you have the class and property path.
Prefer the declarative constraints below; reach for inline Rego only when they cannot express the rule.

## Dialect features beyond `propertyConstraints`

| Feature | Use |
| --- | --- |
| `if:` / `then:` | Guard optional context: "only when a server exists", "only when the type is `string`". An absent guard passes silently. |
| `and:` / `or:` / `not:` | Combine constraint blocks. `not: { or: [...] }` expresses a deny-list. |
| `atLeast:` / `atMost:` / `exactly:` | `{count: N, validation: {...}}`: "at least N children satisfy this". Example: operations with a `200` response need a `429` response. |
| `containsSome` / `containsAll` | List membership, for example a tag allowlist. |
| `nested:` | Constrain the node at the end of a path. Only on the **last** property. |
| `"@type"` | A property inside `nested`/`propertyConstraints`; compare with `containsSome` on full class IRIs. |
| `${{name:default}}` | Parameter placeholder in a constraint value. Published rulesets resolve it to the inline default. It is substituted only in constraint arguments (counts, lengths, `pattern`, `in`, `containsSome`, `datatype`, min/max comparators), **never** in `message` or Rego. |
| `{{prefix.prop}}` | Message template. It reads a property of the **target node only**, and renders empty when the node lacks it (for example `{{apiContract.method}}` on AsyncAPI 3). Write messages that still read correctly when the value is blank. |
| `aspect:` | Optional list tagging a rule with a conformance aspect (catalog, documentation, instances, specification). Seen in published rulesets that validate project or instance data. |
| `regoModule` / `rego_extensions` | Shared Rego helpers; see below. |

A rule with no `message` reports the generic text `Validation error`. Write a fix-oriented message for
every rule.

## Absent values

- `in`, `not`, `pattern` and length constraints all **pass** when the property is missing. Pair them
  with `minCount: 1` when the value is required.
- Inside `or:`, put `minCount: 1` in **each** branch. An `or` of `minLength` branches never fires on an
  absent field.
- Count lists instead of testing presence when emptiness must fail (`minCount: 1`, not just "exists").
- A rule targeting a class that does not exist in the document produces no node, so it silently passes.
  Add a separate presence rule (for example "API declares at least one server").

## Regex

- `pattern` is **unanchored**. `pattern: http` also matches `https`; `pattern: 2.+` matches far more than
  intended. Anchor with `^...$`.
- Write regexes in single-quoted YAML so backslashes are not double-escaped.

## Spec kinds sharing one model

- One `apiContract.*` rule covers RAML, OAS 2, OAS 3 and AsyncAPI. Test each kind you claim to support,
  and add fixtures for AsyncAPI 3.x and gRPC separately when they are in scope.
- **OAS 2 servers are synthesized** from `host`, `basePath` and `schemes`. A URL rule on
  `apiContract.Server` can miss a bare `schemes: [http]`. Target `apiContract.WebAPI` and combine
  `apiContract.server / core.urlTemplate` with `apiContract.scheme` using `or:`.
- Built-in prefixes you can use without declaring them: `apiContract`, `core`, `doc`, `shacl`, `shapes`,
  `security`, `data`, `apiExt`, `sourcemaps`. Declare anything else.
- A prefix namespace may end in any path fragment, not only `#`. That makes snake_case names and names
  containing `.`, `-` or `/` reachable (see the constraint table in `agent-asset-domains.md`).
- Vendor extensions: `apiExt.<extension> / data.<key> / data.value`. Booleans compare as strings
  (`in: ["false"]`).
- Examples may live behind `$ref`. Detect them through source maps, including declared elements, or
  rules on `components/*` produce false positives. Skip virtual (duplicated) parameter nodes when
  counting examples.

## Inline Rego

Use it for set differences, uniqueness across operations, counting thresholds, ordering, cross-node
correlation, or regex extraction.

- Contract: `$node` is the target node; assign `$result` a boolean (`true` = conforming); `$message`
  is optional.
- Helpers: `find`, `collect`, `collect_values`, `nodes_array`, `nested_nodes`, `search_subjects`,
  `values_contains`, `object.get(node, "<full IRI>", default)`. Use **full IRIs** inside Rego.
- A one-element JSON-LD array collapses to a bare object; check `is_array` before indexing.
- Rules that need the whole graph read `input["@ids"]` and `input["@types"]`; references are
  `{"@id": ...}`.
- A Rego evaluation error can still print a passing summary. Read the report itself, not the summary line.
- Publishing rejects Rego that uses `http.send`, `walk`, `opa.runtime`, `rego.parse_module` or
  `net.lookup_ip_addr`.
- Inline Rego breaks validation in the standalone MCP manifest domain (see `agent-asset-domains.md`),
  but works for API-spec, Agent Network and instance rules.

## Severity and messages

| Severity | Use for |
| --- | --- |
| `violation` | Security, structural or format correctness; anything that must block. Only `violation` results make an API nonconformant. |
| `warning` | Documentation, metadata, style; missing recommended data. Default for new rules. |
| `info` | Recommendations. |

- Messages: one short sentence that says how to fix it. `documentation:` carries rationale, scope
  ("applies to ...") and exemptions. `examples: {valid, invalid}` must use the rule's **own** target spec.
- Rule IDs: kebab-case, named for the check (`*-required`, `*-should-be-*`, `no-*`, `*-before-*`).
- Keep rule IDs unique across rulesets that are attached to the same profile; a duplicate ID makes
  results ambiguous.

## Rules that must not over-reach

- Do not require a property the schema marks optional.
- Exempt versions that cannot have the field (for example a card type without `capabilities`) with an
  `if/then` instead of failing it.
- When two rules could report the same defect, make one tolerate what the other reports.
