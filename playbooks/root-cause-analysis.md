# Root Cause Analysis

Use this workflow to investigate an incident, regression, or unexpected system behavior. It is designed to help an engineer organize evidence and test hypotheses, not to manufacture a single confident explanation from incomplete records.

## Inputs to provide

- Incident window and timezone, affected services, symptoms, impact, and detection source
- Timeline, alerts, logs/traces/metrics (sanitized), dashboards, and relevant change/deploy history
- Known-good and failing examples, reproduction steps, and affected/unaffected cohorts
- Architecture/dependency context, runbooks, mitigations, and recovery actions
- Relevant PRs, configuration or schema changes, feature-flag state, and test/CI output
- Constraints: unavailable telemetry, data retention limits, privacy rules, and incident severity policy

## Prompt

```text
Help produce an evidence-driven root cause analysis for the supplied incident. Do not make changes, assign blame, or claim causality from correlation alone.

1. Normalize all timestamps to the incident's stated timezone and preserve original timestamps where useful. If timezone is unknown, mark it UNKNOWN.
2. Build a timeline from supplied evidence. Cite each event with its source and timestamp; distinguish observed events from recollections or inferred sequence.
3. Define impact using evidence: affected users/requests/data/services, duration, and known limits. Mark unmeasured impact UNKNOWN.
4. Generate multiple plausible hypotheses. For each, list supporting evidence, contradicting evidence, confidence, and the next discriminating check. Do not collapse to one cause prematurely.
5. Examine recent changes, dependencies, load, data, configuration, feature flags, and operational actions only where supported by evidence. Separate trigger, contributing conditions, detection gaps, and recovery factors.
6. Mark all unverified statements ASSUMPTION. Do not use “root cause” for a hypothesis that has not been tested. Note missing telemetry and plausible alternatives.
7. Recommend corrective and preventive actions that are specific, owned only when an owner is supplied, and verifiable. Distinguish immediate mitigation from long-term prevention. Avoid blame-oriented language.
8. Do not expose secrets or reproduce sensitive customer data. Use redacted identifiers and aggregates.

OUTPUT
## Incident summary — what happened, impact, duration, and confidence
## Scope and evidence reviewed — systems, sources, gaps
## Timeline — time | event | evidence/source | observed vs inferred
## Hypotheses — table: hypothesis | supporting evidence | contradicting evidence | confidence | next check
## Causal analysis — confirmed trigger (or “not established”), contributing conditions, detection/recovery factors
## Assumptions and unknowns — numbered, with validation method
## Actions — table: priority | action | expected risk reduction | verification/acceptance evidence | owner (if known) | due date (if supplied)
## Lessons and follow-up — process/system improvements, avoiding blame

Use calibrated language. State “cause not established” if the evidence does not distinguish hypotheses.
```

## Worked example

**Scenario:** At 10:05 UTC, checkout errors increased shortly after a deployment. Error rate returned to baseline after traffic was shifted away at 10:31 UTC. A database connection-pool change and a downstream latency alert occurred in the same period; no trace samples were supplied.

**Illustrative output excerpt:**

> **Incident summary:** Checkout error rate increased from 0.3% to 8% between 10:05 and 10:31 UTC based on the supplied dashboard snapshot. Traffic was shifted away at 10:31 UTC and the dashboard returned to baseline by 10:38 UTC. Impacted transaction count is UNKNOWN.
>
> | Hypothesis | Supporting evidence | Contradicting evidence | Confidence | Next check |
> |---|---|---|---|---|
> | Reduced connection-pool capacity caused checkout request failures | Deployment record at 10:02 UTC includes a pool-size change; errors began at 10:05 UTC. | No pool saturation metrics or traces were supplied. | Medium | Compare pool utilization, wait time, and connection timeout counts before/after deployment. |
> | Downstream latency exhausted checkout request deadlines | Downstream latency alert overlaps the incident window. | No per-dependency timeout or trace evidence supplied. | Medium | Correlate downstream latency and checkout timeout logs by request ID. |
>
> **Causal analysis:** Cause is **not established**. The timing supports both hypotheses but does not distinguish them. The traffic shift is a recovery action associated with the return to baseline; it does not prove which condition caused the incident.
>
> **Assumption A1:** The dashboard uses a consistent request denominator throughout the incident. Verify dashboard query and sampling configuration.
>
> **Immediate action:** Preserve sanitized pool metrics, dependency latency, and deployment configuration for the incident window; verify whether the revert or traffic shift changed either variable.

Example is fictional. Follow incident policy and preserve evidence according to organizational retention and privacy requirements.
