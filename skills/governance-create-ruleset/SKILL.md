---
name: governance-create-ruleset
description: Use when creating a new MuleSoft API Governance ruleset from a specification data schema, the ruleset development CLI, or AMF Validation Profile modeling language; not for Rust PDK gateway policies.
---

# Create a governance ruleset

## Choose the path

1. Identify the API type, specification(s), target metadata, rule intent, and expected pass/fail examples. Check Exchange first for an existing ruleset. If one nearly fits, use `governance-customize-ruleset` instead.
2. To derive a starter from a specification's data schema, inspect the spec and select actual type names from the output:

   ```sh
   anypoint-cli-v4 governance:api:inspect path/to/api.yaml
   anypoint-cli-v4 governance:ruleset:init --types TypeOne,TypeTwo --name=my-ruleset path/to/data-schema
   ```

   `init` takes a **data schema**, not the API specification itself. Inspect first to find suitable types. Treat its output as a starting point, not a finished ruleset.
3. For completely new rules, follow the [AMF Rulesets tutorial](https://github.com/aml-org/amf-custom-validator/blob/develop/docs/validation_tutorial/validation.md) and use the open-source [`@aml-org/ruleset-development-cli`](https://www.npmjs.com/package/@aml-org/ruleset-development-cli), or author Validation Profile YAML with the AMF modeling language. Do not invent target classes or property paths; use `governance-author-from-intent` for CLI-assisted discovery.
4. Keep a representative conforming and nonconforming API fixture for each rule. Run `governance-validate-ruleset` before publishing. Keep the YAML and a short explanation of each rule together in the ruleset project. Give each rule a fix-oriented `message`, `documentation`, and inline `examples: {valid, invalid}`. A workable repo layout is `ruleset.yaml`, `exchange.json` (bump `version` on every rule change, because Exchange versions are immutable), `fixtures/good/`, `fixtures/bad/<rule-id>/`, and `scripts/check.sh`.

## Source Ref

- [Creating custom governance rulesets](https://docs.mulesoft.com/api-governance/create-custom-rulesets)
- [Creating completely new custom rulesets](https://docs.mulesoft.com/api-governance/custom-rulesets-new)
- [API Governance full documentation](https://docs.mulesoft.com/api-governance/llms-full.txt)
- Snapshot: 2026-10-09
