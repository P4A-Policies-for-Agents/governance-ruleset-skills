---
name: governance-apply-rulesets
description: Use when scoping published MuleSoft API Governance rulesets to APIs with governance profiles, previewing draft conformance, choosing ruleset versions, or investigating profile matching.
---

# Apply rulesets through governance profiles

Governance profiles **validate** matching APIs and track conformance; they do not attach or enforce gateway policies. Only API Governance or organization administrators may create, edit, or delete profiles. Confirm the scope and obtain approval before remote profile mutations.

1. Identify the ruleset's Exchange GAV (`group-id/asset-id/version`) and the API population. Prefer a pinned version for controlled rollout (a pinned profile never moves and needs a manual edit to pick up a fix); `latest` is resolved when the profile is saved and then follows newer ruleset versions. Check the API type, tags, categories, and instance environment filters. **No criteria targets all APIs in Exchange.** Multiple `tag`/`category` values must all match; multiple `scope`/`env-*` values match any. See [profile-reference.md](profile-reference.md) for the full filter grammar, limits, notifications, and conformance states.
2. Prefer the Governance console: create a profile, choose rulesets and narrow filter criteria, inspect the preview, then **Save as a draft**. Review draft conformance before activating; draft notifications are disabled. Activation exposes conformance outside the draft view and can notify owners.
3. CLI automation creates an **active** profile immediately, not a draft. Use it only after explicit approval of that effect and the complete filter set:

   ```sh
   anypoint-cli-v4 governance:api:evaluate --criteria 'tag:pilot,scope:rest-api'
   anypoint-cli-v4 governance:profile:create 'Pilot Standards' group-id/asset-id/1.0.0 --criteria 'tag:pilot,scope:rest-api' --description 'Pilot API standards'
   anypoint-cli-v4 governance:profile:list --output json
   anypoint-cli-v4 governance:profile:info profile-id
   ```

   `evaluate` predicts matching rulesets; it does not create a profile. CLI `--criteria` accepts `scope` (`rest-api`, `async-api`, `http-api`), `tag`, `category`, `env-type`, and `env-id`. `env-type` or `env-id` limits results to APIs with instances. Use real Exchange tags/categories; do not rely on the console's small preview as an exhaustive list. The CLI exits 0 on several failures (including `profile:create`), so read the printed message instead of trusting the exit code.
4. Review conformance in the draft/active profile or the Governance validation report. Results arrive asynchronously, so re-check after a delay. A nonconformant API has at least one `violation`-severity result in a ruleset; warnings and info never make it nonconformant. **Not Validated** means no ruleset applies to that aspect (or no profile includes the API); Pending and Failed are separate states. Fix specification issues in the API project and republish its new version; check Exchange documentation/catalog data or API Manager for other aspects. Activate a reviewed draft only with approval.

## Source Ref

- [Applying rulesets to identified APIs](https://docs.mulesoft.com/api-governance/create-profiles)
- [Monitoring API conformance](https://docs.mulesoft.com/api-governance/monitor-api-conformance)
- [Finding and fixing conformance issues](https://docs.mulesoft.com/api-governance/find-and-fix-conformance-issues)
- Behavior details in `profile-reference.md`: governance plugin 1.1.4
- Snapshot: 2026-10-09
