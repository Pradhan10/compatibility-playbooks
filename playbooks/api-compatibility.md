# API Compatibility Review

Use this workflow for changes to REST, RPC, GraphQL, or SDK-facing interfaces. It is intended for repository-aware assistants and engineers reviewing an API change before merge or release.

## Inputs to provide

Provide as many as are available; explicitly mark anything unavailable:

- Change description, PR diff, base and proposed revisions
- Current and proposed API contract (for example OpenAPI, protobuf, GraphQL schema)
- API versioning and compatibility policy
- Known server implementations, clients, generated SDKs, gateways, and downstream consumers
- Deprecation policy, traffic/usage evidence, and supported client versions
- Relevant tests, CI results, release/deployment constraints, feature flags, and rollback plan

## Prompt

```text
You are assisting with an enterprise API compatibility review. Analyze the supplied change and repository context. Do not edit files or claim that the change is safe merely because a contract diff appears clean.

OBJECTIVE
Determine whether the proposed API change can break existing callers, generated clients, gateways, integrations, or operational workflows. Identify evidence, uncertainty, and the validation needed before release.

METHOD
1. Establish the reviewed change and comparison baseline. If either is unclear, state that limitation.
2. Inspect relevant API definitions, implementation, callers, generated clients, tests, routing/gateway configuration, and versioning/deprecation documentation available in context.
3. For every material claim, cite evidence precisely: file path and symbol/operation, contract location, command output, commit, or supplied artifact. Separate direct evidence from inference.
4. Mark every inferred but unverified condition as ASSUMPTION. Mark unavailable information UNKNOWN and say how to verify it. Do not infer that a caller does not exist because repository search found none.
5. Assess request and response compatibility separately, including requiredness, types, formats, enum values, defaults, nullability, headers, status codes, error shape, pagination, ordering, idempotency, authentication, and semantics where relevant.
6. Consider generated SDK source/binary compatibility, gateways/validators, old and new server versions, mixed client versions, deprecation windows, and rollback. Scope findings to evidence actually available.
7. Prefer deterministic checks where available (contract diff, SDK generation/compilation, consumer contract tests). Do not invent test results.
8. Recommend the smallest actionable validation or mitigation for each finding.

SEVERITY
- BLOCKER: evidence indicates existing supported clients can fail or data/operations can be corrupted, and no safe mitigation is established.
- HIGH: likely material break for a supported consumer or deployment state; needs resolution or explicit owner acceptance before release.
- MEDIUM: plausible break or important uncertainty requiring validation or a documented decision.
- LOW: limited or unlikely impact; track as an improvement.

OUTPUT (use these headings)
1. Review scope — change, baseline, contracts and repositories inspected; include gaps.
2. Verdict — Compatible / Compatible with conditions / Likely breaking / Inconclusive. Give a one-paragraph evidence-based rationale.
3. Findings — table with Severity, Confidence (High/Medium/Low), Consumer or surface, Change, Evidence, Impact, Required action. If none, say no issues were found in inspected evidence; do not claim universal safety.
4. Compatibility matrix — request, response, errors/status codes, generated clients, gateway/validators, mixed-version rollout, deprecation/rollback; each marked Pass / Risk / Unknown with evidence.
5. Assumptions and unknowns — numbered, with owner or verification method when known.
6. Validation plan — exact commands/tests to run if discoverable, otherwise named test/check to add; expected evidence and responsible owner if known.
7. Release conditions — explicit conditions for proceeding, or “None identified from supplied evidence.”

Use concise, reviewable language. Do not present assumptions as facts or give legal/compliance approval.
```

## Worked example

**Scenario:** `GET /v1/accounts/{id}` previously returns `status` as a string. A change replaces it with `state`, adds `status` only when a compatibility flag is enabled, then plans to remove `status` next quarter.

**Illustrative output excerpt:**

> **Verdict: Compatible with conditions**
>
> | Severity | Confidence | Surface | Change | Evidence | Impact | Required action |
> |---|---|---|---|---|---|---|
> | HIGH | High | Existing REST clients | `status` is removed by default | `services/accounts/openapi.yaml`, `AccountResponse.status`; handler `AccountController.getAccount` serializes `state` unconditionally. No default enabling compatibility was found in inspected config. | Preserve `status` by default for the supported v1 contract, or provide evidence that all supported callers have migrated and obtain API owner approval. |
>
> **Assumption A1:** The compatibility flag is off in production. Verify the deployed configuration and flag targeting before rollout.
>
> **Unknown U1:** External consumers are not represented in the inspected repository. Confirm usage with API gateway analytics or consumer registry; repository search cannot establish their absence.
>
> **Validation:** Run the OpenAPI breaking-change check against the base revision, generate/compile supported SDKs, and add a contract test asserting the v1 response includes `status` during the migration window.

This example is fictional. Adapt severity to the service's documented support and compatibility policy.
