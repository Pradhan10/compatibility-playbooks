# Database and Migration Compatibility Review

Use this workflow for relational or document schema changes, data migrations, index changes, and changes that require coordinated application/database rollout. Focus on mixed application versions and recoverability.

## Inputs to provide

- PR diff, migration scripts, ORM/model changes, and target database/version
- Schema before/after and migration framework/configuration
- Read/write paths, batch jobs, replicas, CDC consumers, and reporting queries
- Table size, traffic, lock/latency constraints, data retention, and backup/restore policy
- Deployment order, application version overlap, feature flags, backfill strategy, and rollback requirements
- Migration/test output and production change-management constraints

## Prompt

```text
Review the supplied database/schema change for compatibility and safe rollout in an enterprise system. Do not edit files, execute destructive operations, or claim recoverability without evidence.

EVIDENCE METHOD
1. Identify database engine/version, schema baseline, proposed migration, application versions, and rollout model from supplied context. Mark missing facts UNKNOWN.
2. Inspect migration code, ORM/models, reads/writes, jobs, CDC/replication consumers, indexes/constraints, tests, and operational scripts available in context.
3. Cite evidence for each claim. Mark inferred conditions ASSUMPTION; repository search cannot establish absence of out-of-repository clients or operators.
4. Analyze the transition, not only the final schema: old app + old schema, old app + new schema, new app + new schema, and new app + old schema where rollback can produce that state.
5. Review additive/expand-contract sequencing, nullable/default semantics, backfill correctness/idempotency, dual reads/writes, constraints, index build behavior, locks, transaction size, replication lag, and data volume where relevant.
6. Assess rollback realistically: whether schema changes are reversible, whether writes under the new application can be read by the old version, and how migrated data would be restored. Do not call a down migration a safe rollback unless data loss/semantics have been assessed.
7. Propose non-production validation and monitoring signals. Do not run a migration or destructive command.

SEVERITY
BLOCKER = likely data loss/corruption or outage with no established recovery. HIGH = likely deployment incompatibility, unacceptable locking/latency, or unsafe rollback. MEDIUM = plausible risk or key operational unknown. LOW = limited impact or improvement.

OUTPUT
## Scope and evidence
## Verdict — Compatible / Compatible with conditions / Likely breaking / Inconclusive
## Findings — table: Severity | Confidence | App/schema state | Evidence | Failure mode | Mitigation/action
## Transition matrix — old app/old schema, old app/new schema, new app/new schema, new app/old schema (if rollback is contemplated); Pass/Risk/Unknown with evidence
## Migration properties — additive/expand-contract, backfill, idempotency, locking, constraints/indexes, replication/CDC; findings or Unknown
## Assumptions and unknowns — numbered with verification path/owner when known
## Validation and monitoring plan — checks, test data/scale, expected evidence, signals and stop conditions
## Rollout/rollback plan — ordered phases and data/recovery constraints; explicitly state what cannot safely be rolled back

Use the database engine's documented behavior and supplied operational constraints. Do not invent lock behavior, execution time, backup status, or rollback guarantees.
```

## Worked example

**Scenario:** A service adds nullable `customer_id`, deploys code that writes it, backfills existing rows, then intends to make the column non-null. The table has roughly 80 million rows; the database engine and online index/constraint behavior were not supplied.

**Illustrative output excerpt:**

> **Verdict: Compatible with conditions; migration safety is inconclusive**
>
> | Severity | Confidence | State | Evidence | Failure mode | Action |
> |---|---|---|---|---|---|
> | HIGH | Medium | New app with old schema | New code writes `customer_id`; migration is a separate deployment step in the supplied plan. | Application rollout before the column exists can fail writes. | Deploy additive nullable-column migration first; verify it completed before enabling application writes. |
> | HIGH | Medium | Backfill and constraint | Backfill targets about 80M rows; no batching, throttling, or engine details supplied. | Long transaction, lock contention, replication lag, or partial completion may affect service. | Test a batched, resumable backfill at representative scale; define lag/latency stop thresholds before production. |
> | UNKNOWN | — | `NOT NULL` transition | Engine/version and online constraint semantics are absent. | Constraint validation may block writes or require a table scan. | DBA/platform owner to confirm engine-specific behavior and maintenance window requirements. |
>
> **Assumption A1:** Existing rows can be mapped to a valid `customer_id`. Verify against source data and quantify unmappable rows before enforcing `NOT NULL`.
>
> **Rollback constraint:** Once the new application writes values or relies on the column, reverting application code may be possible only if the old version tolerates the added column and the data semantics remain valid. Dropping the column would destroy data and is not a safe rollback without an approved recovery plan.

Example is fictional; validate migration behavior on the actual database engine, version, data volume, and topology.
