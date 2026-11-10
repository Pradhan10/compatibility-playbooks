# Example: Expand-Contract Database Migration

This fictional example demonstrates compatibility across application versions and a large-table migration.

## Scenario

The Payments service plans to replace `account_ref` with `account_id` on a table containing approximately 60 million rows. The team proposes a three-release expand-contract migration. The database is PostgreSQL, but exact version, lock budget, replication topology, and backup recovery evidence must still be confirmed.

## Input artifacts

- Current schema and migration scripts
- Application reads/writes, scheduled jobs, and CDC consumers
- Table size and traffic/replication constraints
- Deployment order and supported application-version overlap
- Backfill implementation, idempotency behavior, and progress metrics
- Rollback policy and tested backup/restore or forward-recovery plan

For this example, assume supplied artifacts show:

1. Release A adds nullable `account_id`; old code ignores the column.
2. Release B writes both columns and reads `account_id` with fallback to `account_ref`.
3. A resumable job backfills `account_id` in batches and records a checkpoint.
4. Release C reads only `account_id`; a later migration removes `account_ref`.
5. A test verifies backfill idempotency, but no production-scale lock/lag test is supplied.

## AI review findings

| Severity | Confidence | State | Evidence | Failure mode | Action |
|---|---|---|---|---|---|
| MEDIUM | High | Old app + expanded schema | Release A only adds a nullable column; old code does not reference it in the supplied diff. | No direct incompatibility found in inspected code; external jobs remain unverified. | Search job/CDC/reporting repositories and confirm no schema-sensitive consumers. |
| HIGH | Medium | Backfill at production scale | Backfill batches and checkpoints, but no production-scale lock/replication test is supplied for 60M rows. | Excessive lock time, replication lag, or load could affect service. | Run representative-scale rehearsal; define batch throttling and stop thresholds for latency and replica lag. |
| HIGH | High | Release C and rollback | Release C removes fallback reads; dropping `account_ref` is planned later. | Rolling back to a version that requires `account_ref` after it is dropped may fail. | Keep the old column through the rollback window; explicitly define the point after which rollback requires forward recovery. |
| MEDIUM | Medium | Dual writes | Both columns are written during Release B, but conflict resolution/reconciliation rules are not specified. | Divergent values can produce inconsistent reads or make backfill/retry outcomes ambiguous. | Define source-of-truth and repair behavior; add a consistency metric/check before switching reads. |

## Risk assessment

**Verdict: Compatible with conditions.** The staged plan supports old/new application overlap in principle, and the backfill is resumable. Safety at the target scale and rollback boundary are not yet evidenced.

**Assumption A1:** All writers are covered by Release B dual-write logic. Verify background jobs, admin tools, and CDC ingestion paths.

**Unknown U1:** Production lock and replication impact is unknown until a representative rehearsal is complete.

## Recommended action

Proceed only after confirming all writers and consumers, running a scale-representative migration rehearsal, and defining monitored thresholds for lock duration, write latency, and replication lag. Keep both columns until the rollback window closes and data consistency is verified. Document the irreversible contract point when `account_ref` is dropped; use forward recovery after that point unless a tested restore plan proves otherwise.
