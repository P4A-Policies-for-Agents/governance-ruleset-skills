# Governance profile reference

Behavior details for `governance-apply-rulesets`. The console is authoritative; confirm CLI support
with `governance:profile:create --help` for your plugin version.

## Filter criteria

A filter is comma-separated `key:value` pairs. Split happens at the first colon; a key with no value
is an error, and an unknown key returns a 400 that lists the valid keys.

| Key | Meaning | Multiple values |
| --- | --- | --- |
| `tag`, `category` | Exchange tags / categories | **All** must be present (AND) |
| `scope` (alias `source`) | API type: `rest-api`, `async-api`, `http-api` by default | Any matches (OR) |
| `env-type`, `env-id` | Environment of an API instance; limits results to APIs with instances | Any matches (OR) |
| `status` | Asset lifecycle status | Any matches (OR); ignored for per-API profiles |
| `no-instances` | Only APIs without instances | boolean |
| `managed` | Managed (externally fed) profiles | boolean |

- Different keys combine with AND. An empty filter targets all assets; with no `scope`, `rest-api` is
  assumed.
- Other asset types (for example MCP, agent, GraphQL, gRPC) are accepted as `scope` values only when the
  organization's deployment enables them. Do not promise that profiles evaluate those types until you
  have seen a profile evaluate one.

## Ruleset versions

- A ruleset reference is `group-id/asset-id/<version|latest>`.
- `latest` is resolved when the profile is saved; the save fails if the asset has no versions. When a
  newer version is published, profiles on `latest` move to it and re-validate. Publishing an older
  version does not move them.
- A pinned version **never moves**. A pinned profile needs a manual edit to pick up a fixed ruleset.
- Exchange versions are immutable and results are cached per ruleset version, so pin when you need
  repeatable results and re-publish under a new version to change behavior.
- A ruleset may appear once per profile.

## Draft, active, managed

- A draft is planned and validated, so its preview shows real results, but it is excluded from active
  conformance and from notification recipients. Activating an already-active profile is an error.
- The CLI and the console create **active** profiles by default; use the console's draft route to
  preview first.
- Managed profiles are fed by externally posted reports and cannot be edited or deleted.
- Creating a profile that has the same status, filter and ruleset set as an existing one is rejected
  (409). Name and description lengths are capped; trial subscriptions also cap profile and ruleset
  counts.
- Tuning: start with a few APIs, keep few rulesets per profile, and use one profile per related API set.

## Notifications

Default: enabled, sent on failure, by email to the asset Contact and Publisher. Recipient types are
Contact, Publisher, Governor and Others (a capped list of email addresses). Slack and Teams channels
exist only in the newer organization-scoped API, not the CLI. Notifications fire only for nonconformant
results, once for each newly failing ruleset on an API version. CLI flags: `--notify-publisher`,
`--notify-contact`, `--notify-others`, `--notify-off`.

## Per-profile customization (alternative to cloning)

The organization-scoped API can customize a ruleset inside one profile without cloning it: move rules
between severity levels, set string parameters for parameterized rules, and attach remediations. A
severity list that is absent keeps the ruleset's own assignment; an empty list clears that level (which
disables those rules); a non-empty list sets exactly those rules. Unknown rule IDs are skipped. The CLI
does not expose this; use `governance-customize-ruleset` to clone instead.

## Conformance results

- Evaluation is asynchronous. After publishing a ruleset, editing a profile, or publishing an API
  version, expect a delay and re-check later; there is no synchronous guarantee.
- Per ruleset and API the status is Conformant, NonConformant, Pending, Failed or Revalidate. Failed
  validations are retried and stay Pending meanwhile. The console shows aspects (Global, Specification,
  Catalog info, Instances, Documentation); an aspect with no applicable ruleset shows **Not Validated**.
  Failed appears as "Not Available".
- Overall precedence: NonConformant, then Pending, then Failed, then Conformant.
- **Only `violation` results make an API nonconformant.** Warnings and info are reported but leave it
  conformant. Moving a rule from `violation` to `warning` therefore changes conformance.
- The asset page's Conformance tab shows "No errors found" for any conformant report, hiding its
  warnings and info. The governance console's per-ruleset table shows warning and info counts, so
  check warning- and info-only test fixtures there. The CSV export lists only pass/fail per ruleset.
- Fixing the spec requires republishing a new API version; instance data is fixed in API Manager;
  catalog data in Exchange.

## CLI behavior to remember

- `governance:profile:create`, `delete` and `api:evaluate` can print an error and still exit 0. Read the
  printed message.
- `--output json` (`-o`) exists only on `profile:create`, `profile:info` and `profile:list`, and emits an
  array of arrays, not objects.
- `profile:update`, `profile:delete`, `profile:list` and `profile:info` exist besides `create`.
- Unauthenticated runs exit 2 with "No authentication mechanism was provided".
- CSV export of conformance is available in the console from profiles and governed-API views.
