---
name: governance-author-from-intent
description: Use when translating a natural-language API Governance requirement into an AMF Validation Profile YAML ruleset with Anypoint CLI governance:ruleset authoring and discovery commands.
---

# Author rules from intent

The CLI's `generate` command returns **context and instructions**, not a finished verified ruleset. Never publish its output without authoring, validation, and tests. Check that the installed governance plugin supports the authoring commands (the [MuleSoft authoring skill](https://dev-portal.mulesoft.com/skills/mule-development/author-governance-ruleset/SKILL.md) requires version 1.0.20 or newer); ask before changing an installed plugin.

1. Ask for the domain (API spec, MCP, API project, governance, or community), affected spec type, rule severity, a passing example, and a failing example. If a spec is available, provide it as context:

   ```sh
   anypoint-cli-v4 governance:ruleset:version
   anypoint-cli-v4 governance:ruleset:generate 'Every operation must have a description' --api-spec path/to/openapi.yaml
   ```

2. Resolve ambiguous human terms into model paths. The resolver's aliases are for discovery only; write its **canonical** class and property path in YAML. Run `domains` to choose a domain, and keep a single `specKind` per ruleset (do not mix `api-spec` with `mcp`). Discover classes, properties, and constraints rather than guessing:

   ```sh
   anypoint-cli-v4 governance:ruleset:domains
   anypoint-cli-v4 governance:ruleset:resolve 'operation description'
   anypoint-cli-v4 governance:ruleset:classes --domain api-spec
   anypoint-cli-v4 governance:ruleset:properties apiContract.Operation
   anypoint-cli-v4 governance:ruleset:constraints --type scalar
   ```

3. Write the Validation Profile 1.0 YAML against the discovered model. Start with `#%Validation Profile 1.0` on line 1; declare `profile`, `validations`, and a `propertyConstraints` entry for each rule, and assign every rule to `violation`, `warning`, or `info`. Use only constraints compatible with the **last** property in a nested path (`scalar`, `node`, `scalarArray`, or `nodeArray`), and declare non-default namespace prefixes. Use the [AMF validation tutorial](https://github.com/aml-org/amf-custom-validator/blob/develop/docs/validation_tutorial/validation.md) for the actual dialect structure. If an editor supports it, `governance:ruleset:completions path/to/ruleset.yaml --offset <cursor-offset> --line-text '    targetClass: '` provides contextual suggestions.
4. Run both checks, then test against a passing and a failing API (see `governance-validate-ruleset`):

   ```sh
   anypoint-cli-v4 governance:ruleset:validate-authoring path/to/ruleset.yaml
   anypoint-cli-v4 governance:ruleset:validate path/to/ruleset.yaml
   ```

   `validate-authoring` checks class, path, constraint compatibility, and severity; it exits 1 on errors. `validate` checks dialect conformance. Neither proves the rule detects the intended defect. Run `governance:ruleset:simplify path/to/ruleset.yaml`, review its printed YAML before replacing the source, and revalidate any rewritten file. Show the resulting ruleset and test outcomes to the user.

## Source Ref

- [Anypoint CLI 4.x governance authoring commands](https://docs.mulesoft.com/anypoint-cli/latest/api-governance)
- [Creating completely new custom rulesets](https://docs.mulesoft.com/api-governance/custom-rulesets-new)
- [MuleSoft ruleset authoring skill](https://dev-portal.mulesoft.com/skills/mule-development/author-governance-ruleset/SKILL.md)
- Snapshot: 2026-10-05
