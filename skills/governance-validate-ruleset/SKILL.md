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

   The first command checks Validation Profile dialect conformance. The second checks model classes, paths, constraints, and severities when available in your CLI version. Fix every error before continuing. Both need a real `.yaml` file path (process substitution fails with "not a supported type"), and neither is a full check: a file with no `#%Validation Profile 1.0` header can still print "Ruleset is valid"; `in` on a scalar and other incompatible constraints are only warnings; severity entries naming rules that do not exist, rules left out of every severity list (they never run), and undeclared prefixes are not flagged; and `validate` accepts a made-up class. Check the header, severity lists, and prefixes yourself, and confirm every class and path with `classes`/`properties`. A globally installed CLI may bundle an older plugin without these helpers, so run `governance:ruleset:version` first. For authoring changes, run `governance:ruleset:simplify path/to/ruleset.yaml`, inspect its output, and repeat both validations if you apply the simplification. Keep one spec kind per ruleset unless you deliberately mix spec-graph rules with instance rules and test each side.
2. Prepare an API project folder or ZIP with a **passing** and a **failing** spec variant. Validate each with the local ruleset:

   ```sh
   anypoint-cli-v4 governance:api:validate path/to/passing-api-project --rulesets path/to/ruleset.yaml
   anypoint-cli-v4 governance:api:validate path/to/failing-api-project --rulesets path/to/ruleset.yaml
   ```

   `governance:api:validate` takes an API **project folder/ZIP** with `exchange.json` at its top level, or a remote API GAV with `--remote`; do not assume an arbitrary raw spec file is accepted. Remote rulesets (`--remote-rulesets`) must be `group/asset/version` and are cached locally, so a republished version can look unchanged. **The exit code is 0 even when the API does not conform.** Gate on the parsed `Conforms:` line and the per-result `Severity:`/rule ID, not the exit status. Only `violation` results set `Conforms: false`; a warning or info leaves it true, so assert on rule IDs and severities, not just `Conforms:`. Check that the intended rule ID, severity, location, and message appear **only** for the failing case. Separate functional parse errors from conformance violations. When combining `exchange.json` ruleset dependencies with `--rulesets` or `--remote-rulesets`, avoid accidental duplicate validation.
3. An empty result is not a pass until you rule out a skipped document. Script these checks (for example in `scripts/check.sh`) and fail on any of them:
   - The output contains `legacy mode` (`APB validation failed, falling back to legacy mode: <reason>`, or `Legacy project descriptor found`). Read the reason after the colon: the `exchange.json` classifier is wrong or unknown, the descriptor is old-style, a dependency fetch failed, or a rule panicked the validator. The CLI may then report nothing and exit cleanly, or print a differently shaped result (`Spec conforms|does not conform with Ruleset`), so parse both shapes or fail.
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
   Fixtures must themselves satisfy the spec's schema (a wrong key or a disallowed character makes the test misleading). Cover OAS 2 as well as OAS 3 when the rule targets servers or schemes, and give AsyncAPI 3.x, gRPC, and catalog/instance rules their own fixtures. Rules on catalog, documentation or instance data cannot use a spec file; test them with graph dumps, for example with the open-source `ruleset-development-cli test` (stored expected reports; regenerate after each rule change) or `acv validate ruleset.yaml graph.jsonld`. A summary line can say "pass" after a Rego evaluation error, so read the report itself.
5. Record the commands, CLI and plugin versions, fixture paths, and outcomes for reviewers. A dialect-valid YAML can still target the wrong API model property. Publishing runs only a compile check, so nothing here is re-verified at upload.

## Source Ref

- [Validating custom rulesets](https://docs.mulesoft.com/api-governance/custom-rulesets-validate-and-publish)
- [Anypoint CLI 4.x API validation reference](https://docs.mulesoft.com/anypoint-cli/latest/api-governance#governance-api-validate)
- [MuleSoft ruleset authoring skill](https://dev-portal.mulesoft.com/skills/mule-development/author-governance-ruleset/SKILL.md)
- Snapshot: 2026-10-09
