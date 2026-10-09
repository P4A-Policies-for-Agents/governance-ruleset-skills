# MCP and A2A asset domains: verified behavior

Facts the authoring helpers do not tell you. Each was confirmed by running fixtures, not by
`validate-authoring` or `validate`. Both of those accepted rules that were wrong at runtime.

## Fixture shape (both domains)

`governance:api:validate` takes a project folder: the document plus an `exchange.json`.

```json
{"main": "mcp-metadata.json", "name": "good", "groupId": "fixtures", "assetId": "good",
 "version": "1.0.0", "classifier": "mcp-metadata", "descriptorVersion": "1.0.0"}
```

| Asset | `classifier` | Minimum plugin |
| --- | --- | --- |
| MCP server manifest | `mcp-metadata` | 1.0.20 |
| A2A v1.0 Agent Card | `a2a-v1-card` | 1.1.4 |
| A2A v0.3 Agent Card | `a2a-card` | — |

An unknown or wrong classifier (for example `mcp`) makes the CLI print `falling back to legacy
mode` and report no rule findings. That empty output looks exactly like a pass. The document
is also checked against Anypoint's JSON schema, and schema failures show up as
`example-validation-error`. That result is a fixture bug, not a rule finding.

To get a newer plugin without touching the global CLI, use a pinned local install and invoke
its binary:

```json
{"private": true, "dependencies": {"anypoint-cli-v4-public": "1.6.27"},
 "overrides": {"mulesoft-anypoint-cli-governance-plugin": "1.1.4"}}
```

## MCP manifests (`mcp-metadata`)

- Write `core.*` property paths, such as `core.description`. `mcp.*` paths pass both static
  checks but match the wrong node at runtime.
- Declare `prefixes: { mcp: http://anypoint.com/vocabs/mcp# }`. Without it the validator panics
  at runtime, and neither static check notices.
- Per-parameter rules can't be expressed. `JsonSchemaProperty` is never instantiated, and
  `core.properties` is an opaque `dynamic` value. Inline `rego` breaks validation. Rules can
  reach tools, tool input/output schemas, annotations, resources, prompts, prompt arguments
  and server-level fields.

## A2A v1.0 Agent Cards (`a2a-v1-card`)

- The card is a generic graph. Each field is `core.<jsonName>`, and each nested object's class
  is `core.<jsonKey>`: a skill is `core.skills`, an interface is `core.supportedInterfaces`.
- The document root hangs off the project node:
  `api.Project → api.contract / doc.encodes → card`.
- `core.encodes`, `core.provider`, `core.capabilities` and `core.flows` also exist in MCP
  manifests and v0.3 cards. A card-level rule must target `api.Project` and be guarded by the
  classifier. These findings are reported on the asset and have no source line.

  ```yaml
  prefixes:
    api: http://anypoint.com/vocabs/api#
    catalog: http://anypoint.com/vocabs/digital-repository#
  validations:
    card-name-required:
      targetClass: api.Project
      if:
        propertyConstraints:
          catalog.classifier:
            in: [a2a-v1-card]
      then:
        propertyConstraints:
          api.contract / doc.encodes / core.name:
            minCount: 1
            minLength: 1
  ```

- Don't nest `propertyConstraints` (`nested:`) under an `api.Project` path. Doing so breaks
  validation of every document.
- For per-element rules (each skill, each interface), use the element class as `targetClass`.
  A flat path such as `… / core.skills / core.tags` is evaluated across all skills together,
  so one skill with tags satisfies the rule for all of them. `validate-authoring` reports
  `Invalid targetClass` for `core.skills` and similar classes. `governance:ruleset:validate`
  accepts them, and the fixtures prove they work. Allowlist exactly those classes in your
  harness.
- The schema accepts snake_case aliases (`icon_url`, `protocol_version`) and stores them under
  their own names; it does not normalize them. A camelCase rule therefore needs a separate
  "alias must not be used" rule.

## Constraint patterns

| Need | Pattern | Why |
| --- | --- | --- |
| Value from a set, and required | `minCount: 1` + `in: [...]` | `in` alone ignores missing values. |
| Field must be absent | `maxCount: 0` on its path | — |
| Class instance must not exist | `minCount: 1` + `maxCount: 0` on any property of that class | Can never pass, so any instance fails. |
| Path to a snake_case field | Prefix whose namespace ends with the leading words: `snakeIcon: http://a.ml/vocabularies/core#icon_`, then path `snakeIcon.url` | A compact IRI can't contain `_`. `core.icon_url` panics with `is not in compact form`, which the CLI reports as legacy mode. |
| Rule on a snake_case wrapper object under a user-chosen key | `targetClass: <prefix>.<rest>`, for example `snakeApiKeySecurity.scheme` | The wrapper is its own class even under a map key. |

A rule with N failing paths reports N results for one document. Count findings per rule ID,
not raw results.
