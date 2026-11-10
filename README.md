# Compatibility Playbooks

**Portable, evidence-driven AI workflows for reviewing API, event, and database compatibility—and the release risks around them.**

For staff and platform engineers, architects, tech leads, and SREs working across services, contracts, and production rollouts. Use these Markdown playbooks with GitHub Copilot, OpenAI Codex, Cursor, Claude Code, or another assistant that can inspect the context you provide.

They help turn a change into a reviewable account of **what could break, what evidence supports that concern, what remains unknown, and what to verify before release**. They complement contract tests, schema checks, migration tests, operational knowledge, and human decisions; they do not certify safety.

## Start here

1. Pick the workflow for the change: [API](playbooks/api-compatibility.md), [event/message](playbooks/event-compatibility.md), [database/migration](playbooks/database-compatibility.md), [release readiness](playbooks/release-readiness.md), or [root cause analysis](playbooks/root-cause-analysis.md).
2. Provide the playbook's requested inputs: diff, before/after contract or schema, relevant callers, deployment constraints, and check results.
3. Ask your assistant to inspect available context and cite paths, symbols, commits, or supplied artifacts for findings.
4. Verify material claims with deterministic checks and system owners. Keep the resulting review with the PR, incident, or release record.

**Example:** Before changing a Kafka payload, provide the schema diff, producer and consumer code, schema-registry policy, and rollout plan. The event workflow asks the assistant to assess old/new producer-consumer combinations, retained-message replay, and unknown external consumers—then return findings with evidence, confidence, and validation actions.

> **Data handling:** Do not provide secrets, credentials, customer data, or restricted production logs unless your organization's policy explicitly permits it. Prefer sanitized examples and aggregates.

## Workflows

| Playbook | Use it for |
|---|---|
| [API compatibility review](playbooks/api-compatibility.md) | REST, RPC, GraphQL, and SDK-facing contract changes |
| [Event compatibility review](playbooks/event-compatibility.md) | Kafka, queues, webhooks, CDC, and message payload changes |
| [Database compatibility review](playbooks/database-compatibility.md) | Schema changes, data migrations, and mixed application/schema versions |
| [Release readiness review](playbooks/release-readiness.md) | Deployment order, migration, observability, and recovery readiness |
| [Root cause analysis](playbooks/root-cause-analysis.md) | Incidents and regressions where evidence and competing hypotheses matter |

See [worked examples](examples/) for fictional enterprise scenarios and sample findings.

## What makes these useful

- **Evidence first:** Findings should cite source artifacts and distinguish observation from inference.
- **Unknowns stay visible:** Missing information is marked, with a verification path, rather than guessed.
- **Enterprise transitions:** Workflows consider callers, producers/consumers, mixed versions, migration order, rollback limits, and operations.
- **Tool-neutral Markdown:** Copy the prompt into your assistant or adapt it to your team's repository instructions.
- **Practical outputs:** Findings, risk, validation steps, and release conditions are structured for review and follow-up.

## Scope and limitations

These are decision-support workflows. Assistants can only reason over context they can access; an undiscovered consumer or undocumented constraint can change the conclusion. A clean prompt review does not replace deterministic contract/schema checks, tests, production telemetry, or accountable human approval. Treat an “all clear” as limited to the evidence inspected.

The repository focuses on compatibility and adjacent release/incident workflows. It does not provide an automated reviewer, guarantee compatibility, or connect to Jira, GitHub, or production systems.

## Contributing

Focused improvements and realistic, sanitized examples are welcome. See [CONTRIBUTING.md](CONTRIBUTING.md), the [Code of Conduct](CODE_OF_CONDUCT.md), and [Security policy](SECURITY.md). Please do not submit secrets, proprietary incident records, or customer data.

## License

MIT. See [LICENSE](LICENSE).
