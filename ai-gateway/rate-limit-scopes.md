# AI Gateway Rate Limits: What Should Each Limit Control?

This is a product design framework, not a provider-specific implementation guide. All numeric examples are illustrative.

For definitions of RPM, TPM, concurrency, and the difference between quota and throughput, start with the [metrics primer](model-api-metrics.md).

## 1. Problem: one limit cannot answer every question

A gateway exposes one model API backed by several upstream endpoints. Each user can access several models through multiple API keys.

Where should the gateway enforce rate limits?

The answer depends on the problem. An upstream limit protects a provider quota. A user limit controls one consumer's traffic. A model limit controls traffic to one logical model API. These limits can apply to the same request without being interchangeable.

**Separate upstream capacity from consumer allowances.** The gateway must respect available capacity even when a user has allowance remaining. Spare upstream capacity does not entitle a user to exceed their allowance.

## 2. Separate the scopes

Here, a **model API** is the logical model exposed by the gateway. An **upstream** is a destination the gateway can route requests to.

| Scope | What it controls | Recommended role |
| --- | --- | --- |
| Upstream quota pool | Traffic consuming a provider or serving quota, possibly shared by several endpoints. | Respect the upstream's limits. |
| Model API, across all users | Aggregate traffic to one logical model, across its upstreams. | Optional operator-defined policy cap. |
| User, across all models | Aggregate traffic from one user, across all their API keys. | Shared consumer allowance. |
| User, per model | One user's traffic to a particular model. | Optional model-specific cap. |
| API key | Traffic using one credential. | Optional workload-specific restriction within the owner's allowance. |

These are gateway-side controls. The gateway operator defines the allowances; users do not have to configure them in their own API management pages. A product can offer delegated controls, but those must not loosen the operator's policy.

Use the identity that actually owns the allowance. For an individual service, that may be a user. For a team platform, it may be a project or organization. Production capacity should not depend on which employee created a key.

The rest of this article uses **user** as the quota owner.

## 3. Learn upstream limits; do not assume them

Start with the provider's account console, quota API where available, or contract. Documentation explains defaults and quota rules, but may not reflect the account's assigned limits.

Response headers can provide remaining quota and reset information where supported. Rate-limit errors and `Retry-After` can guide backoff, but an error alone may not identify which shared quota was exhausted.

Testing can validate sustainable throughput. It is not authoritative quota discovery: results depend on workload, latency, concurrent consumers, and provider load. Tests also consume capacity and may affect other traffic.

Record the RPM, TPM, and concurrency limits with their accounting rules. Also record quota ownership: the account, project, model, region, or other pool sharing each allowance.

For a provider-specific example, [OpenAI's rate-limit documentation](https://developers.openai.com/api/docs/guides/rate-limits) explains organization and project limits, shared model-family limits, and request accounting. Those upstream rules are evidence to collect, not a requirement to copy the provider's scopes into gateway user policies.

Two endpoints using one provider account may share a quota. Treating them as independent pools would overstate capacity. If another application consumes the same quota outside the gateway, local counters alone cannot know the full remaining allowance.

When limits are unknown, show that explicitly and require an operator-selected fallback policy. Do not silently interpret missing data as unlimited capacity.

## 4. Use upstream capacity automatically, without inventing a model allowance

Suppose upstream A allows 100 RPM and B allows 200 RPM.

If their quotas are independent and routing can use both, their theoretical combined capacity is 300 RPM. If both consume one shared 200 RPM pool, it is not 300 RPM. If B fails, A cannot absorb more than its own available capacity.

Token limits and request sizes further constrain which requests can fit. A fixed RPM sum cannot represent all these conditions.

**The default model API policy can be "Use available upstream capacity."** The gateway tracks known quota availability and routes each request to an eligible destination. It queues within a defined bound or rejects the request when no eligible destination has capacity. Provider feedback still matters because availability is not perfectly predictable.

An administrator does not need to enter a duplicate model-level cap merely to protect upstream quotas.

An additional model-wide cap is a separate choice. For example, an operator might restrict aggregate demand for a costly model. It does not replace a spending budget or reserve capacity unless those mechanisms are explicitly implemented.

On the model API page, distinguish:

- **Model policy cap:** optional; default "No additional cap."
- **Upstream capacity:** known quotas and current availability, with shared pools identified.

If a combined figure is shown, label it as an estimate with its assumptions, not as guaranteed throughput.

## 5. Give each user a shared allowance

Enforcing only per-key limits lets a user multiply their allowance by creating more keys.

Instead, all keys belonging to a user consume shared user counters. Add per-key restrictions only when useful, such as separating a test workload from production.

Support both RPM and TPM at both user scopes:

| User scope | RPM controls | TPM controls |
| --- | --- | --- |
| Across all models | Total request rate from the user. | Total token rate from the user. |
| Per model | Request rate to a particular model. | Token rate on that model. |

For example:

| Policy | RPM | TPM |
| --- | --- | --- |
| User total | 100 | 100,000 |
| User on model A | 20 | 30,000 |

The user could send 20 requests to A and 80 to B in the same window, provided token usage also fits both applicable TPM limits. They could not send 30 requests to A, even with unused global allowance.

**Per-model and per-key caps are ceilings, not additional allowances or reservations.** Every request must satisfy every applicable policy. A user-wide cap also prevents switching models from multiplying the total allowance.

Cross-model token totals are a traffic measure, not a common measure of cost. Models can have different tokenizers and prices. Keep cost budgets separate.

## 6. Define enforcement behavior, not just configuration fields

The gateway can enforce shared limits across applications because it sees their aggregate traffic. A client SDK can help applications back off, but cannot be the authoritative enforcer of a shared allowance.

Configuration belongs in the control plane. Live enforcement belongs on the request path, with coordinated counters across gateway instances. Giving every instance its own full allowance would multiply the effective limit.

RPM and TPM also need explicit accounting rules. Define the window and burst behavior. A token bucket and a fixed one-minute window do not admit exactly the same traffic.

For gateway user quotas, one policy is to count input plus output tokens. Output length is unknown at admission, so reserve an allowance based on estimated input and an enforced output bound. Reconcile against actual usage afterward. Large reservations can reduce utilization; that is a product tradeoff, not free precision.

Upstream accounting must follow the provider's rules, which may differ from the gateway's user policy. Define how failed requests, retries, cancellations, and missing usage affect counters. Never treat missing usage as zero by default. An upstream retry is another upstream attempt, even if the gateway counts the logical user request only once.

When a policy rejects a request, return a clear rate-limit error identifying the relevant scope and dimension without exposing other users' information. Distinguish a user quota rejection from unavailable upstream capacity. Give retry guidance when it can be determined. If the limiter itself is unavailable, use an explicit failure policy rather than silently dropping enforcement.

## 7. PM decisions

1. **Who owns the allowance?** Choose user, project, or organization based on the customer model.
2. **Which controls are defaults?** Start with shared owner-wide RPM and TPM. Add per-model caps where differentiated access is needed. Keep per-key restrictions optional.
3. **Is a model cap policy or capacity?** Use upstream-aware admission for capacity. Add a model-wide policy cap only for an explicit operator need.
4. **What happens when a limit is reached or unknown?** Define rejection, bounded waiting, retry guidance, and accounting behavior.

These are design recommendations, not measured outcomes. Validate them against real traffic and track quota rejections separately from upstream throttling and gateway failures.

**Reusable lesson:** assign allowances to consumers, respect capacity at its actual shared scope, and require every request to satisfy all applicable limits.

[Back to the AI gateway module](README.md)
