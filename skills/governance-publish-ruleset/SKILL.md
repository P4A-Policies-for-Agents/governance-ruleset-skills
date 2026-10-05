---
name: governance-publish-ruleset
description: Use when preparing a validated custom API Governance ruleset and generated documentation for an explicitly approved upload to Anypoint Exchange.
---

# Publish a governance ruleset

Uploading changes remote Exchange state. **Stop for explicit user approval** of the target organization, asset ID, version, visibility, and files before uploading; never infer permission to publish from a request to create or validate a ruleset.

1. Complete `governance-validate-ruleset` and confirm the intended file and rule list:

   ```sh
   anypoint-cli-v4 governance:ruleset:validate path/to/ruleset.yaml
   anypoint-cli-v4 governance:ruleset:info path/to/ruleset.yaml
   ```

2. Generate the associated documentation ZIP:

   ```sh
   anypoint-cli-v4 governance:document path/to/ruleset.yaml path/to/ruleset.doc.zip
   ```

3. Check the selected CLI organization, the proposed `group-id/asset-id/version`, name, description, and generated ZIP. After approval, upload **both** files in one `--files` JSON object:

   ```sh
   anypoint-cli-v4 exchange asset upload group-id/asset-id/1.0.0 --name 'Team Ruleset' --description 'Team API standards' --files='{"ruleset.yaml":"path/to/ruleset.yaml","docs.zip":"path/to/ruleset.doc.zip"}'
   ```

   Omit `--type` when uploading both classifiers; the CLI infers the asset type. If the group ID is omitted, it uses the selected organization. Use `--status development` only when a development asset is intended; the default is published. Confirm the new asset is a **ruleset** in Exchange and inspect its rendered documentation and version.
4. On 409, check for an existing asset/version and select a **new** version or asset ID; do not overwrite by guesswork. On type-inference errors, check `--files` classifiers, remove conflicting `--type`, and update or remove a stale `exchange.json` in the project. An asset published as the wrong type cannot be re-published under that same ID as a ruleset.

## Source Ref

- [Validating and publishing custom rulesets](https://docs.mulesoft.com/api-governance/custom-rulesets-validate-and-publish)
- [Troubleshooting Anypoint CLI](https://docs.mulesoft.com/api-governance/cli-commands-troubleshoot)
- Snapshot: 2026-10-05
