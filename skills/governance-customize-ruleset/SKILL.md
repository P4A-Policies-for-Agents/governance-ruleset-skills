---
name: governance-customize-ruleset
description: Use when cloning a local or Exchange-published MuleSoft API Governance ruleset to change rule severity or disable specific rules while preserving a reviewable before/after comparison.
---

# Customize an existing ruleset

1. Identify the source ruleset file or Exchange GAV (`group-id/asset-id/version`). Confirm you have permission to use and modify the source. Read its rules before changing anything:

   ```sh
   anypoint-cli-v4 governance:ruleset:info path/to/source.yaml
   anypoint-cli-v4 governance:ruleset:info group-id/asset-id/version --remote
   ```

2. Record the rule IDs to change. `--error`, `--warning`, and `--info` move rule IDs to those severity sections; `--remove` **deletes** the rules (they are not commented out), and the output is re-rendered, so source comments are lost. A rule ID that does not exist prints nothing and still exits 0, so always diff against the source. Clone to a **new** file and choose a distinct title and description:

   ```sh
   anypoint-cli-v4 governance:ruleset:clone path/to/source.yaml 'Team Ruleset' 'Customized standards' --warning=rule-id > team-ruleset.yaml
   anypoint-cli-v4 governance:ruleset:clone group-id/asset-id/version 'Team Ruleset' 'Customized standards' --remote --remove=rule-id > team-ruleset.yaml
   ```

3. Run `governance:ruleset:info team-ruleset.yaml` and compare rule IDs and severities with the original; review the YAML to ensure only intended changes. Do not treat a removed rule as merely downgraded, and remember that only `violation` results make an API nonconformant: moving a rule from `--error` to `--warning` can turn nonconformant APIs conformant. Run `governance:ruleset:validate-authoring team-ruleset.yaml` after any manual changes; `simplify` prints a preamble before the YAML on stdout, so strip it before saving, review the result, and revalidate if you apply it. To change severities for one governance profile only, without a new asset, see per-profile customization in `governance-apply-rulesets`' `profile-reference.md`.
4. Validate the new ruleset and exercise positive/negative API examples with `governance-validate-ruleset`. Publish with `governance-publish-ruleset` only after review, using your **own** Exchange asset ID and version.

## Source Ref

- [Modifying published rulesets](https://docs.mulesoft.com/api-governance/custom-rulesets-modify)
- [Validating and publishing custom rulesets](https://docs.mulesoft.com/api-governance/custom-rulesets-validate-and-publish)
- Snapshot: 2026-10-09
