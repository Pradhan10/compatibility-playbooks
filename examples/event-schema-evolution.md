# Example: Event Schema and Meaning Change

This fictional example shows why schema acceptance alone does not establish end-to-end compatibility.

## Scenario

The Orders service publishes `order.total` as integer minor currency units (for example, 1234 means 12.34 in the currency's minor unit). A proposed event change converts the field to a decimal major-unit amount and adds `REFUND_PENDING` to the `order.state` enum. The schema registry is configured for backward compatibility. One batch consumer can lag by up to 36 hours.

## Input artifacts

- Before and after event schema, serializer configuration, and registry compatibility mode
- Producer mapping and consumer deserialization/processing code
- Consumer inventory, lag/retention/replay policy, and DLQ behavior
- Deployment order, event versioning conventions, and rollback plan
- Contract tests and schema compatibility check results

For this example, assume supplied artifacts show:

- Producer changes `total` from integer minor units to decimal major units without renaming the field.
- `finance/reconcile` multiplies `total` by 100 before storing minor units.
- `warehouse/OrderState` uses an exhaustive enum mapping without a fallback branch.
- Registry checks validate the serialized schema but are not shown to validate semantic units.

## AI review findings

| Severity | Confidence | Surface | Evidence | Failure mode | Action |
|---|---|---|---|---|---|
| BLOCKER | High | `finance/reconcile` | Consumer multiplies `total` by 100; producer changes the same field to major units. | Consumer may store 100 times the intended amount, corrupting reconciliation. | Preserve the old unit contract or introduce an explicitly versioned field/event with coordinated consumer migration. |
| HIGH | High | `warehouse/OrderState` | Enum mapping has no fallback; new `REFUND_PENDING` value is not handled in the inspected code. | Deserialization or processing may fail when the new value arrives. | Add explicit handling or safe unknown-value behavior and test it before publishing the new value. |
| MEDIUM | Medium | Lagging/replaying consumers | One consumer may process messages up to 36 hours late; retention and replay tests were not supplied. | Old consumer versions may see new payloads after deployment or replay. | Test old consumer against new payload and replay path; coordinate drain/upgrade or dual-publish strategy. |

## Risk assessment

**Verdict: Likely breaking.** The field's meaning changes while its name remains stable, creating a high-confidence semantic incompatibility even if the schema registry accepts the shape. The new enum value presents a separate consumer risk.

**Assumption A1:** `finance/reconcile` processes this event version in production. Verify deployment configuration and subscription records.

**Unknown U1:** External consumer handling of `REFUND_PENDING` is not established. Confirm with the event ownership registry and consumer teams.

## Recommended action

Do not publish the unit change in place. Preserve `total` semantics and add a new versioned field or event with explicit units and conversion rules. Roll out consumer support before producer emission; test old/new producer-consumer combinations and retained-message replay. Require schema compatibility checks plus semantic contract tests. Define monitoring for deserialization failures, DLQ growth, and reconciliation discrepancies, with stop thresholds and an owner before rollout.
