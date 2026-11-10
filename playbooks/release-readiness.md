# Release Readiness Review

Use this workflow before a production release, especially when the change affects contracts, stored data, asynchronous processing, infrastructure, or multiple services. It consolidates evidence and conditions; it is not an approval gate by itself.

## Inputs to provide

- Release/PR summary, changeset, services and owners
- Completed API, event, and database reviews where applicable
- CI, contract, integration, migration, security, and performance results
- Deployment sequence, dependencies, feature flags, canary/gradual rollout plan
- Dashboards, alerts, SLO/error budgets, on-call and support ownership
- Rollback/forward-fix plan, data recovery limits, release window constraints
- Open risks, approvals, and customer/internal communication needs

## Prompt

```text
Assess production release readiness from the supplied release and repository evidence. Do not approve or block a release on behalf of an owner. Do not infer that a control passed because it is normally present in the organization.

1. Establish release scope, services, versions, dependencies, and reviewed changes. Cite source artifacts for material claims.
2. Inspect supplied CI/test results, compatibility reviews, migration plan, deployment configuration, monitoring, operational docs, and ownership information.
3. Separate EVIDENCE, INFERENCE, ASSUMPTION, and UNKNOWN. Mark unavailable proof UNKNOWN and identify the artifact or owner needed to resolve it.
4. Check readiness across: change/compatibility risk; build and tests; deployment order and mixed-version behavior; data migration/backfill; flags and staged rollout; observability and alert response; rollback/forward recovery; operational ownership; security/compliance gates if relevant; customer/support communication if relevant.
5. Evaluate whether rollback is feasible after data or event changes. Identify irreversible actions and forward-recovery needs.
6. Produce actionable conditions with named owners only when supplied. Avoid generic “monitor closely” advice: specify signal, threshold/decision rule, and response when evidence exists; otherwise state what must be defined.

OUTPUT
## Scope and evidence reviewed
## Readiness verdict — Ready / Ready with conditions / Not ready / Inconclusive. Explain that this is an evidence-based assessment, not release authorization.
## Gate matrix — Area | Status (Pass/Risk/Unknown/Not applicable) | Evidence | Gap/action | Owner (if known)
## Top risks — ordered by severity, each with evidence, impact, mitigation, and confidence
## Deployment sequence — phases, dependencies, compatibility windows, and stop/rollback points; mark unknowns
## Observability and response — signals, thresholds, dashboard/alert links or Unknown, owner and response
## Recovery plan — rollback versus forward fix, data consequences, prerequisites, and unproven assumptions
## Open decisions and release conditions — explicit decision/owner/evidence needed; do not fabricate approvals

If evidence is insufficient for a reliable assessment, choose Inconclusive and list the minimum evidence needed. Never state “safe to ship” as an unconditional guarantee.
```

## Worked example

**Scenario:** A release adds an event field and database column. API contract CI passed; the event consumer inventory is incomplete. A canary is planned, but no rollback decision is documented for rows written with the new column.

**Illustrative output excerpt:**

> **Readiness verdict: Not ready (conditions unresolved)**
>
> | Area | Status | Evidence | Gap/action | Owner |
> |---|---|---|---|---|
> | API contract | Pass | Supplied CI run `#842` reports OpenAPI compatibility check passed against `main`. | Confirm the checked artifact matches the release commit. | Release engineer (as supplied) |
> | Event consumers | Unknown | Consumer inventory is marked incomplete in release ticket `REL-221`. | Complete ownership/usage check and validate old-consumer behavior before enabling publication. | Unknown |
> | Database recovery | Risk | New application writes the added column; no rollback procedure is attached. | Decide rollback vs forward-fix and document treatment of rows written by the new version; test the recovery path. | Service owner (not named) |
>
> **Deployment condition:** Do not enable the event field or make new writes dependent on the column until consumer compatibility and recovery behavior have evidence and an accountable owner.
>
> **Canary observability:** The supplied material does not identify a dashboard, threshold, or response for consumer failures or database write errors. Define those before rollout; do not treat “watch the canary” as a complete control.

Example is fictional. Release decisions remain with the organization's designated approvers.
