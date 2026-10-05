# governance-ruleset-skills

Task-oriented skills for creating, checking, publishing, and applying **MuleSoft API Governance
rulesets**. Like [omni-gateway-pdk-skills](https://github.com/P4A-Policies-for-Agents/omni-gateway-pdk-skills),
each skill lives in `skills/<name>/SKILL.md` with a trigger description and a documented workflow.
These skills concern AMF Validation Profile rulesets, **not** Rust/PDK gateway policies. A
governance profile validates matching APIs against rulesets; it does not enforce gateway policies.

## Skills

| Skill | Use it to |
| --- | --- |
| [governance-create-ruleset](skills/governance-create-ruleset/SKILL.md) | Choose a creation path and scaffold a new ruleset from a schema or the AMF modeling language. |
| [governance-customize-ruleset](skills/governance-customize-ruleset/SKILL.md) | Clone a published/local ruleset and adjust rule severity or disable rules. |
| [governance-author-from-intent](skills/governance-author-from-intent/SKILL.md) | Turn a natural-language requirement into a checked Validation Profile using CLI authoring helpers. |
| [governance-validate-ruleset](skills/governance-validate-ruleset/SKILL.md) | Check ruleset syntax and behavior against a representative API specification. |
| [governance-publish-ruleset](skills/governance-publish-ruleset/SKILL.md) | Generate docs and upload a validated ruleset to Anypoint Exchange. |
| [governance-apply-rulesets](skills/governance-apply-rulesets/SKILL.md) | Scope rulesets to APIs with draft or active governance profiles and inspect conformance. |

Install the `skills/` directory into a skills location your agent supports, or read individual
playbooks directly. Commands shown are examples with placeholders; confirm credentials, organization,
asset identifiers, and effects before running commands that change remote state.

## Sources

Derived from the [API Governance documentation index](https://docs.mulesoft.com/api-governance/llms.txt),
[full documentation](https://docs.mulesoft.com/api-governance/llms-full.txt), and the
[Anypoint CLI 4.x governance reference](https://docs.mulesoft.com/anypoint-cli/latest/api-governance).
Reviewed on 2026-10-05. The documentation is updated independently of this repository; check the
linked page before relying on version-sensitive commands.
