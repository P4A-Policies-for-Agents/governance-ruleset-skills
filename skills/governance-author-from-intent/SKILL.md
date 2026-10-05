---
name: governance-author-from-intent
description: Use when translating a natural-language API Governance requirement into an AMF Validation Profile YAML ruleset with Anypoint CLI governance:ruleset authoring and discovery commands.
---

# Author rules from intent

The CLI's `generate` command returns **context and instructions**, not a finished verified ruleset. Never publish its output without authoring, validation, and tests.

1. Ask for the domain (API spec, MCP, API project, governance, or community), affected spec type, rule severity, a passing example, and a failing example. If a spec is available, provide it as context:

   ```sh
   anypoint-cli-v4 governance:ruleset:version
   anypoint-cli-v4 governance:ruleset:generate 'Every operation must have a description' --api-spec path/to/openapi.yaml
   ```

2. Resolve ambiguous human terms into model paths. Discover classes and property types rather than guessing names:

   ```sh
   anypoint-cli-v4 governance:ruleset:domains
   anypoint-cli-v4 governance:ruleset:resolve 'operation description'
   anypoint-cli-v4 governance:ruleset:classes --domain api-spec
   anypoint-cli-v4 governance:ruleset:properties apiContract.Operation
   anypoint-cli-v4 governance:ruleset:constraints --type scalar
   ```

3. Write the Validation Profile 1.0 YAML against the discovered model. Start with `#%Validation Profile 1.0`, give each constraint a validation field, and assign each rule to `violation`, `warning`, or `info`. Use the [AMF validation tutorial](https://github.com/aml-org/amf-custom-validator/blob/develop/docs/validation_tutorial/validation.md) for the actual dialect structure. If an editor supports it, `governance:ruleset:completions path/to/ruleset.yaml --offset <cursor-offset> --line-text '    targetClass: '` provides contextual suggestions.
4. Run both checks, then test against a passing and a failing API (see `governance-validate-ruleset`):

   ```sh
   anypoint-cli-v4 governance:ruleset:validate-authoring path/to/ruleset.yaml
   anypoint-cli-v4 governance:ruleset:validate path/to/ruleset.yaml
   ```

   `validate-authoring` checks class, path, constraint compatibility, and severity; it exits 1 on errors. `validate` checks dialect conformance. Neither proves the rule detects the intended defect. Optionally run `governance:ruleset:simplify path/to/ruleset.yaml` and review its printed YAML before replacing the source.

## Source Ref

- [Anypoint CLI 4.x governance authoring commands](https://docs.mulesoft.com/anypoint-cli/latest/api-governance)
- [Creating completely new custom rulesets](https://docs.mulesoft.com/api-governance/custom-rulesets-new)
- Snapshot: 2026-10-05
