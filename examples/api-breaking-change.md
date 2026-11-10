# Example: Removing a Required API Response Field

This fictional example shows the **API Compatibility Review** workflow applied to a breaking change. Values and service names are illustrative.

## Scenario

The Billing API proposes to remove `invoiceStatus` from `GET /v1/invoices/{id}` and replace it with `status`. The endpoint remains versioned as v1. The PR description says all callers should migrate, but provides no usage report or deprecation notice.

## Input artifacts

- Base and proposed OpenAPI documents
- Handler and response DTO diff
- Internal client references and generated SDK compile results, if available
- API versioning/deprecation policy
- Gateway traffic report or consumer inventory
- Deployment sequence and rollback plan

For this example, assume the supplied artifacts show:

- Base contract requires `invoiceStatus: string`.
- Proposed contract removes `invoiceStatus` and adds required `status: string`.
- The handler emits only `status` after deployment.
- Internal search finds two callers still reading `invoiceStatus`.
- No external consumer inventory or usage data is available.

## AI review findings

| Severity | Confidence | Surface | Evidence | Impact | Action |
|---|---|---|---|---|---|
| HIGH | High | Two internal callers | `reporting/InvoiceExport.java` and `legacy-sync/InvoiceMapper.ts` read `invoiceStatus`; proposed handler no longer serializes it. | Callers may fail deserialization or lose the status value. | Preserve the old field during a migration window or migrate and verify both callers before removing it. |
| HIGH | Medium | External consumers | No gateway usage report or external consumer inventory was supplied. | Existing clients may break even after internal callers migrate. | Obtain API usage evidence and follow the documented deprecation policy; repository search cannot establish that external consumers are absent. |
| MEDIUM | High | Generated SDKs | Proposed schema removes a required property and introduces another required property. | Generated clients may have source or runtime compatibility impact. | Run supported SDK generation and compile/compatibility checks against the base and proposed contracts. |

## Risk assessment

**Verdict: Likely breaking.** Evidence confirms internal callers use the removed field. The external impact is unknown, not zero. Keeping the endpoint at `/v1` increases the expectation that existing supported clients continue to work, subject to the service's documented policy.

**Assumption A1:** The two callers are deployed against this API in production. Verify against service ownership/deployment records.

**Unknown U1:** Whether external consumers use `invoiceStatus`. Check gateway analytics or the API consumer registry.

## Recommended action

Keep `invoiceStatus` in the v1 response while adding `status`, unless the API owner has evidence and approval to retire it. Add contract tests for both fields during migration, update internal clients, publish a deprecation timeline, then remove the legacy field only under the service's versioning policy. Run an OpenAPI diff and supported SDK checks in CI. Define how rollback behaves if clients begin relying on `status`.
