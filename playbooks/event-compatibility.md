# Event and Message Compatibility Review

Use this workflow for changes to Kafka topics, queues, webhooks, CDC records, or other asynchronous message contracts. Evaluate schema compatibility and behavior across independently deployed producers and consumers.

## Inputs to provide

- PR diff and before/after event schema or payload examples
- Serialization format and schema registry policy/configuration
- Producer and consumer code, topic/queue configuration, and ownership map
- Consumer lag, replay/retention, retry, DLQ, ordering, and idempotency expectations
- Deployment topology and rollout order; old/new producer and consumer overlap
- Backfill/replay plan, feature flags, monitoring, and rollback procedure

## Prompt

```text
Review the supplied event/message change for backward compatibility in an enterprise system. Do not change code and do not equate schema-registry acceptance with end-to-end safety.

1. Identify the event/topic/queue, serialization format, baseline and proposed schema, and compatibility mode from supplied evidence. If missing, mark UNKNOWN.
2. Inspect producers, consumers, generated models, schema registry settings, tests, deployment manifests, retention/replay behavior, and relevant docs available in context.
3. For every finding, cite the exact evidence and distinguish observed facts, inference, and ASSUMPTION. A repository search with no matches does not prove there are no external consumers.
4. Assess both directions: new consumers reading old retained messages, old consumers reading new messages, and old/new producer-consumer combinations during rolling deployments.
5. Check field additions/removals/renames, requiredness, defaults, null/absent distinction, type/format changes, enum expansion, unknown-field handling, serialization behavior, semantic meaning, ordering, keys/partitioning, headers, and schema references where relevant.
6. Consider consumer lag, replay, retries, DLQ, deduplication/idempotency, retention, data backfills, PII/security classification, and topic migration/dual-publish requirements where evidence makes them applicable.
7. Recommend deterministic schema compatibility checks and producer/consumer contract tests. Never state that checks passed unless output was supplied or run in the available environment.

SEVERITY
BLOCKER = supported consumers may fail, lose/incorrectly interpret messages, or create unrecoverable state. HIGH = likely break during normal or mixed-version operation. MEDIUM = plausible risk or important unknown needing owner validation. LOW = limited impact or improvement.

OUTPUT
## Scope and evidence
## Verdict — Compatible / Compatible with conditions / Likely breaking / Inconclusive
## Findings — table: Severity | Confidence | Producer/consumer or data surface | Change | Evidence | Failure mode | Action
## Compatibility matrix — old producer→old consumer; old producer→new consumer; new producer→old consumer; new producer→new consumer; replayed retained messages. Mark Pass/Risk/Unknown and cite evidence.
## Assumptions and unknowns — label each, include verification method/owner when known
## Validation plan — schema check, contract tests, replay/lag test, deployment simulation, or other justified checks; include expected result
## Rollout and rollback conditions — sequence, monitoring, stop/rollback trigger, and constraints; use Unknown where not established

Limit conclusions to supplied evidence. Do not claim universal safety or substitute for the service's event contract policy.
```

## Worked example

**Scenario:** A Kafka event `invoice.v2` changes `amount` from integer cents to decimal currency units and adds an enum value `PARTIALLY_REFUNDED`. The schema registry is configured for backward compatibility. A batch consumer is known to lag by up to 48 hours.

**Illustrative output excerpt:**

> **Verdict: Likely breaking**
>
> | Severity | Confidence | Surface | Change | Evidence | Failure mode | Action |
> |---|---|---|---|---|---|---|
> | BLOCKER | High | Existing consumers | `amount` changes representation and unit | Proposed schema changes `amount` from integer to decimal; `billing/recon/InvoiceEvent.java` multiplies integer cents by 100 before formatting. | Consumer may reject the payload or interpret the value at the wrong scale, corrupting reconciliation. | Introduce a versioned field/event and migrate consumers with an explicit conversion contract; do not silently change units in place. |
> | MEDIUM | Medium | Enum consumers | New enum value may not be handled | `reporting/InvoiceStatus.java` switches on known values; no default branch found in inspected code. | Strict deserialization or exhaustive handling may fail. External consumers remain UNKNOWN. | Add unknown-value handling or coordinate consumers; query consumer ownership/usage records. |
>
> **Assumption A1:** Schema registry enforcement checks the configured schema rules but does not validate semantic unit changes. Verify the registry's exact compatibility mode and serializer behavior.
>
> **Unknown U1:** It is not established whether all consumers tolerate unknown enum values. Confirm through consumer inventory and contract tests.
>
> **Rollout condition:** Preserve the old event contract until lagging consumers have drained or are proven to support the new version; verify replay behavior before enabling any backfill.

Example is fictional. Schema compatibility modes vary by registry and format; verify the actual configuration and semantics.
