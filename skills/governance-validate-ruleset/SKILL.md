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

   The first command checks Validation Profile dialect conformance. The second checks model classes, paths, constraints, and severities when available in your CLI version. Fix every error before continuing. For authoring changes, run `governance:ruleset:simplify path/to/ruleset.yaml`, inspect its output, and repeat both validations if you apply the simplification. Keep one spec kind per ruleset.
2. Prepare an API project folder or ZIP with a **passing** and a **failing** spec variant. Validate each with the local ruleset:

   ```sh
   anypoint-cli-v4 governance:api:validate path/to/passing-api-project --rulesets path/to/ruleset.yaml
   anypoint-cli-v4 governance:api:validate path/to/failing-api-project --rulesets path/to/ruleset.yaml
   ```

   `governance:api:validate` takes an API **project folder/ZIP**, or a remote API GAV with `--remote`; do not assume an arbitrary raw spec file is accepted. Check that the intended rule ID, severity, location, and message appear **only** for the failing case. Separate functional parse errors from conformance violations. When combining `exchange.json` ruleset dependencies with `--rulesets` or `--remote-rulesets`, avoid accidental duplicate validation.
3. An empty result is not a pass until you rule out a skipped document. Script these checks (for example in `scripts/check.sh`) and fail on any of them:
   - The output contains `falling back to legacy mode`. The `exchange.json` classifier is wrong or unknown, or a rule panicked the validator. The CLI then reports nothing and exits cleanly.
   - The output contains `example-validation-error`. The fixture fails Anypoint's JSON schema, so the fixture needs fixing; this is not a rule finding.
   - The number of findings you parsed differs from the CLI's `Number of results:` line.
   - The governance plugin is older than the version the document's classifier needs. Check this first and fail fast.

   For MCP or A2A classifiers and plugin versions, see `governance-author-from-intent`'s `agent-asset-domains.md`.
4. Make the fixture set prove each rule in isolation:
   - `fixtures/good/` gives 0 findings.
   - Each `fixtures/bad/<rule-id>/` gives exactly one rule ID with its severity. A path that fails on N fields reports N results, so dedupe by rule ID. Use `fixtures/bad/<rule-id>.<variant>/` for each alias or edge case, such as a missing value versus a wrong value.
   - Scope fixtures stay clean: documents of other asset types the ruleset must not touch, and valid cards that omit optional fields.
   - A sibling ruleset's `good` fixture stays clean, which catches rules that are noisy on realistic input.

   Generate the bad fixtures from `good` plus one mutation each, using a small script, instead of editing copies by hand. Hand-edited copies drift and start failing more than one rule. Also lint the YAML in the harness: line-1 header, each rule in exactly one severity list, the severity lists match the `validations:` keys, and the file is a sensible size. `governance:api:validate` signs in to Anypoint, and an occasional 401 is transient, so rerun before debugging.
5. Record the commands, CLI and plugin versions, fixture paths, and outcomes for reviewers. A dialect-valid YAML can still target the wrong API model property.

## Source Ref

- [Validating custom rulesets](https://docs.mulesoft.com/api-governance/custom-rulesets-validate-and-publish)
- [Anypoint CLI 4.x API validation reference](https://docs.mulesoft.com/anypoint-cli/latest/api-governance#governance-api-validate)
- [MuleSoft ruleset authoring skill](https://dev-portal.mulesoft.com/skills/mule-development/author-governance-ruleset/SKILL.md)
- Snapshot: 2026-10-09
