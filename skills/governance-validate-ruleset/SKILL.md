---
name: governance-validate-ruleset
description: Use when checking a custom MuleSoft API Governance ruleset's dialect, authoring model, and actual conformance results against representative API projects before publication.
---

# Validate a governance ruleset

1. Validate the local YAML (or a project folder/ZIP with `exchange.json` pointing to the main ruleset file):

   ```sh
   anypoint-cli-v4 governance:ruleset:validate path/to/ruleset.yaml
   anypoint-cli-v4 governance:ruleset:validate-authoring path/to/ruleset.yaml
   ```

   The first command checks Validation Profile dialect conformance. The second checks model classes, paths, constraints, and severities when available in your CLI version. Fix every error before continuing.
2. Prepare an API project folder or ZIP with a **passing** and a **failing** spec variant. Validate each with the local ruleset:

   ```sh
   anypoint-cli-v4 governance:api:validate path/to/passing-api-project --rulesets path/to/ruleset.yaml
   anypoint-cli-v4 governance:api:validate path/to/failing-api-project --rulesets path/to/ruleset.yaml
   ```

   `governance:api:validate` takes an API **project folder/ZIP**, or a remote API GAV with `--remote`; do not assume an arbitrary raw spec file is accepted. Check that the intended rule ID, severity, location, and message appear **only** for the failing case. Separate functional parse errors from conformance violations. When combining `exchange.json` ruleset dependencies with `--rulesets` or `--remote-rulesets`, avoid accidental duplicate validation.
3. Record the commands, CLI version, fixture paths, and outcomes for reviewers. A dialect-valid YAML can still target the wrong API model property.

## Source Ref

- [Validating custom rulesets](https://docs.mulesoft.com/api-governance/custom-rulesets-validate-and-publish)
- [Anypoint CLI 4.x API validation reference](https://docs.mulesoft.com/anypoint-cli/latest/api-governance#governance-api-validate)
- Snapshot: 2026-10-05
