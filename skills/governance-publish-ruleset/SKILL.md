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

   Exchange runs a **compile-only** check on upload: YAML and structure parse, then profile compile. It does not check that classes or paths exist, that fixtures behave, or that documentation exists. Compile failures you may see: `missing targetClass in validation definition`; inline Rego using `http.send`, `walk`, `opa.runtime`, `rego.parse_module` or `net.lookup_ip_addr`; an undeclared prefix or non-`prefix.term` path (`Term ... not present in context`, `... is not in compact form`). A rule defined under `validations` but absent from `violation`/`warning`/`info` is **silently skipped**, and so is a listed name with no definition; check both lists match before uploading.

2. Generate the associated documentation ZIP. Add `--single-page` for rulesets with many rules; per-rule pages can time out on upload (502):

   ```sh
   anypoint-cli-v4 governance:document path/to/ruleset.yaml path/to/ruleset.doc.zip
   ```

3. Check the selected CLI organization, the proposed `group-id/asset-id/version`, name, description, and generated ZIP. After approval, upload **both** files in one `--files` JSON object, and name the ruleset YAML as `mainFile` in `--properties`:

   ```sh
   anypoint-cli-v4 exchange asset upload group-id/asset-id/1.0.0 --name 'Team Ruleset' --description 'Team API standards' --properties='{"mainFile":"ruleset.yaml"}' --files='{"ruleset.yaml":"path/to/ruleset.yaml","docs.zip":"path/to/ruleset.doc.zip"}'
   ```

   Omit `--type` when uploading both classifiers; the CLI infers the asset type. If the group ID is omitted, it uses the selected organization. Use `--status development` only when a development asset is intended; the default is published. Confirm the new asset is a **ruleset** in Exchange and inspect its rendered documentation and version.

   `mainFile` is the file name of the uploaded ruleset YAML (`ruleset.yaml` here). Without `--properties`, Exchange rejects the upload with 400 `There are missing require properties ... ["mainFile"]`.

   The leading part of each `--files` key is the asset classifier, so keep the ruleset key named `ruleset.yaml` (or `ruleset.zip`). A key such as `my-rules.yaml` yields a different classifier and fails with `no files found with classifier ruleset`. A packaged alternative is a ZIP that holds `ruleset.yaml` plus an `exchange.json` (`"classifier": "ruleset"`, `"main": "ruleset.yaml"`, `"descriptorVersion": "1.0.0"`), uploaded as `ruleset.zip`; then `mainFile` must equal an entry path in the ZIP exactly, case included, or the upload fails with `File not found on zip: <mainFile>`. If the ruleset is kept in several metadata files (asset properties, `exchange.json`), bump the version in all of them together.
4. On a 400 about missing `mainFile`, add the `--properties` flag above. On 409, check for an existing asset/version and select a **new** version or asset ID; do not overwrite by guesswork. On type-inference errors, check `--files` classifiers, remove conflicting `--type`, and update or remove a stale `exchange.json` in the project. An asset published as the wrong type cannot be re-published under that same ID as a ruleset. The Exchange web UI cannot upload ruleset assets, so the CLI is the route. Custom rulesets are not supported by MuleSoft; rule-engine defects belong with the open-source AMF validator project. Remote rulesets are cached under `~/.anypoint_deps` (or `$ANYPOINT_HOME`), so clear a stale entry before concluding a republished version did not change. Consumers that pin a version never see a fix until the profile is edited, so announce new versions.

## Source Ref

- [Validating and publishing custom rulesets](https://docs.mulesoft.com/api-governance/custom-rulesets-validate-and-publish)
- [Troubleshooting Anypoint CLI](https://docs.mulesoft.com/api-governance/cli-commands-troubleshoot)
- Snapshot: 2026-10-09
