# Compatibility Playbooks

Reusable, tool-neutral AI workflows for reviewing backward compatibility and release risk in enterprise software systems.

These playbooks are designed to guide an engineer and an AI coding assistant through evidence gathering, impact analysis, and reviewable recommendations. They do not replace contract tests, schema registries, CI checks, operational knowledge, or human approval.

## Who this is for

- Engineers changing APIs, event contracts, database schemas, or deployment behavior
- Staff and platform engineers reviewing cross-service impact
- Tech leads, architects, SREs, and release owners preparing changes for production
- Teams using GitHub Copilot, OpenAI Codex, Cursor, Claude Code, or another assistant that can inspect supplied repository context

## Quick start

1. Choose the playbook that matches the change.
2. Open it in your coding assistant or copy its **Prompt** section into a session.
3. Supply the requested context: diff, contract/schema versions, relevant callers, deployment constraints, and test results.
4. Ask the assistant to inspect the repository and cite file paths, symbols, commands, or supplied artifacts for each material finding.
5. Verify findings with deterministic checks and the responsible system owners before approving or deploying.

Do not paste secrets, customer data, credentials, or restricted production logs into an assistant unless your organization's policies explicitly allow it.

## Playbooks

| Playbook | Use it when |
|---|---|
| [API compatibility](playbooks/api-compatibility.md) | Changing REST, RPC, or SDK-facing contracts |
| [Event compatibility](playbooks/event-compatibility.md) | Changing Kafka, queue, webhook, or other event payloads |
| [Database compatibility](playbooks/database-compatibility.md) | Changing schemas, data migrations, or application/schema rollout order |
| [Release readiness](playbooks/release-readiness.md) | Assessing deploy, migration, observability, and rollback readiness |
| [Root cause analysis](playbooks/root-cause-analysis.md) | Investigating an incident or unexpected behavior with evidence |

Each playbook contains a copy-ready prompt, required inputs, an output contract, evidence rules, and a worked example. Examples are fictional and are intended to show the expected reasoning and format.

## How these playbooks work

The assistant should:

- Separate **observed evidence**, **inference**, and **assumption**.
- Cite the source for each repository-specific claim using paths, symbols, commit references, test output, or supplied artifacts.
- Mark missing information as **Unknown** and state how to verify it; never fill gaps with plausible-sounding facts.
- Describe impact in terms of producers, consumers, clients, operators, data, and rollout states where applicable.
- Recommend concrete validation steps and identify which decisions require human confirmation.
- Report findings by severity and confidence, without implying that a review proves the system is safe.

## Suggested use in an enterprise repository

Run deterministic checks first where available, such as OpenAPI diff tooling, generated-client compilation, schema-registry compatibility checks, migration tests, and contract tests. Use the playbook to investigate context those checks cannot establish, such as actual caller behavior, mixed-version deployment windows, operational constraints, or rollback feasibility.

Keep the review in the change record (for example, a PR or release ticket). Redact sensitive information and follow organizational data-handling policy.

## Limitations

- An assistant only reasons over context it can access. A missing consumer, undocumented integration, or unprovided deployment constraint can invalidate a conclusion.
- Static compatibility tools and runtime behavior may disagree with documentation; record the discrepancy and ask the relevant owner.
- Compatibility is contextual. A change can be wire-compatible yet break behavior, operational procedures, generated clients, or data recovery.
- These prompts are decision support. They do not certify compatibility, replace tests, or authorize production changes.

## Contributing

Contributions should improve a real review workflow, not merely add role-play instructions. Include:

1. The scenario and intended audience.
2. Required inputs and the evidence the assistant should inspect.
3. An example that demonstrates the expected output (use fictional or safely sanitized data).
4. Known limitations and deterministic checks that should accompany the prompt.

Keep playbooks tool-neutral. Tool-specific installation adapters can be added separately when they preserve the canonical workflow. Never include secrets, customer data, proprietary incident records, or unlicensed source material.

## License

MIT. See [LICENSE](LICENSE).
