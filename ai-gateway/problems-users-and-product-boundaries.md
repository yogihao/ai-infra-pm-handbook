# AI Gateway: Problems, Users, and Product Boundaries

This is a product framework, not a specific implementation. The use cases are illustrative. They do not describe real incidents or measured results.

## 1. Problem: why add a gateway?

An application can call a model service directly. A gateway becomes useful when several applications face the same problems.

| Shared problem | Why a gateway can help |
| --- | --- |
| Repeated provider integrations | Share adapters and failure-handling policies. |
| Scattered credentials and access rules | Control who can use each model and destination. |
| Inconsistent content and data policies | Apply shared **guardrails**, such as sensitive-data checks and content screening. |
| Traffic spikes and competing teams | Apply **rate limits** to protect capacity and keep access fair. |
| Unclear usage and cost ownership | Attribute requests and usage to projects. |

For this module, an **AI gateway** sits between applications and model services. It applies shared access, traffic, and usage policies. Model services may be external APIs or self-hosted endpoints.

Guardrails and rate limits solve different problems. Guardrails check content against defined policies. They may block or redact sensitive data, or flag disallowed content. Rate limits control traffic over time, such as requests or tokens per minute. Concurrency limits cap in-flight requests. These controls can work together.

Neither is a complete guarantee. Content checks can miss harmful content or block legitimate requests. Rate limits do not guarantee a spending cap. Budget controls need separate accounting and enforcement.

The value is more than "one endpoint for many models." It is less repeated work and more consistent controls. Applications should still get the capabilities they need.

## 2. Use cases: who needs what?

Each use case connects a user need to possible gateway behavior. These are design choices, not a required feature list.

| Use case | Who needs it? | Possible gateway behavior | Key design question |
| --- | --- | --- | --- |
| Integrate another model provider | Application developers | Adapt requests, responses, streams, and errors. | Which provider-specific features must remain available? |
| Control model access | Platform and security teams | Check project identity and model permissions before forwarding. | Who manages upstream credentials and approved destinations? |
| Apply shared guardrails | Security teams and application owners | Check inputs and outputs. Block, redact, or flag content by policy. | Which checks need application context? |
| Share limited capacity | Platform operators | Enforce per-project rate and concurrency limits. Return clear rejection errors. | How should capacity be shared during bursts? |
| Handle an unavailable endpoint | Developers and operators | Route to an approved alternative when retrying is safe. | Can the alternative meet quality, data, and cost requirements? |
| Attribute usage and investigate failures | Finance, PMs, and operators | Record project-level usage and request traces. Reconcile available upstream usage. | What needs application-level evidence or billing reconciliation? |

These needs can conflict. A developer may want a new provider immediately. A security team may require approval first. A PM must choose priorities.

Some use cases also have hard limits. Output screening may require buffering, which delays delivery. Chunk-level checks can miss context. Content already delivered cannot be recalled.

A stream can fail after partial output. Retrying may duplicate work or cost. Usage data may also be incomplete. Define these behaviors rather than assuming a gateway makes failures transparent.

## 3. Design layers: where should capabilities live?

**A capability is not the same as a deployment location.** Policy and operational work can live in an application, a shared SDK, or a gateway. Some responsibilities are split across them.

The layers below group responsibilities. They are not mandatory services or a fixed execution order.

| Design layer | Application or SDK placement | Gateway placement | Main tradeoff |
| --- | --- | --- | --- |
| Product workflow | Prompts, retrieval, tool workflows, and task evaluation. | Reusable workflows, if explicitly part of the platform scope. | Sharing reduces duplication but couples the platform to product behavior. |
| Model interface | Native provider SDKs or shared adapters. | A common API with provider adapters. | Local integration exposes new features quickly. Central adapters reduce repeated maintenance. |
| Policy and guardrails | Task-specific permissions and content checks. | Organization-wide access rules and shared screening. | Applications have richer context. A trusted shared path can enforce common rules. |
| Traffic and reliability | User-level limits, task deadlines, and acceptable fallback choices. | Cross-application limits, endpoint routing, and shared retry budgets. | Local control fits each workflow. Shared control sees aggregate demand. |
| Usage and observability | User outcomes, task quality, and business context. | Request traces, upstream usage, and project attribution. | Neither view is complete alone. Link them with stable identifiers. |

For example, an application may choose acceptable fallback models. The gateway can route among them. An application may limit each user's activity. The gateway can enforce a shared project quota.

### Separate policy ownership from enforcement

A security team may define allowed data destinations. An application team may define task-specific restrictions. The gateway can enforce both on the request path.

Configuration belongs to the **control plane**. Live request handling belongs to the **data plane**. These responsibilities may share one deployment. Policy ownership does not depend on where the checks run.

A shared SDK distributes code, but does not guarantee enforcement. Applications may use an old version or bypass it. A trusted application backend can enforce local rules. Organization-wide rules need coverage across all permitted access paths.

### Choose placement deliberately

| Placement | Benefit | Cost |
| --- | --- | --- |
| Application | Rich workflow context and independent releases. | Teams may repeat policy and operational work. |
| Shared SDK | Reuses code without another network hop. | Changes depend on application rollout. Aggregate limits still need shared coordination. |
| Gateway | Central updates and cross-application enforcement. | Adds latency, operating cost, and a shared failure domain. |

These placements can coexist. Centralization moves operational work; it does not remove it. Decide who owns configuration, deployment, incidents, and upgrades.

Avoid overlapping controls without coordination. Application and gateway retries can multiply attempts. Independent per-instance limits cannot enforce an exact global quota without coordination.

Keep application-specific logic close to the application. Centralize controls that need shared visibility or consistent enforcement. For every shared dependency, define failure behavior. A guardrail outage, for example, needs an explicit block-or-allow policy.

Some boundaries remain useful. GPU scheduling, batching, and KV-cache management belong near model execution. Gateway routing may use serving signals, but it does not replace the serving scheduler. Identity, secrets, and billing can remain in supporting systems.

Tool/MCP gateways add execution and authorization concerns. This series starts with model access.

Before building, compare existing API gateways, managed AI gateways, and shared tooling. Choose based on unmet needs and operational ownership.

For a concrete implementation, [Cloudflare's official AI Gateway documentation](https://developers.cloudflare.com/ai-gateway/) describes analytics, caching, rate limiting, retries, and model fallback. It illustrates the category; it does not prescribe this article's product boundaries.

## 4. PM judgment: start with one valuable promise

For teams that need shared controls and usage attribution, a first promise could be:

> Every request on this path has a project owner. It follows approved access, content, and traffic policies. Its usage and failures can be investigated.

Start with project identity, scoped access, secure credentials, and usage visibility. Add traffic limits and the guardrails required by the actual risk. Make rejections clear. Advanced routing and caching can wait unless a real need requires them.

The promise applies only to traffic through the gateway. Applications might bypass it with separate credentials and unrestricted network access. Gateway policy alone cannot govern that traffic.

### Measure the promise, not the feature count

| Dimension | Example measure | Interpretation guardrail |
| --- | --- | --- |
| Adoption | Time to first successful integration; share of eligible traffic using the gateway. | Define eligible traffic and understand why teams bypass it. |
| Reliability | Completion rate; gateway failures versus upstream failures. | Successful response headers do not mean a stream completed. |
| Guardrails | Missed violations and incorrect blocks on labeled test cases. | A high block rate alone does not prove safety. |
| Traffic controls | Rate-limit rejections by project; overload incidents. | Separate expected policy rejections from service failures. |
| Performance | Added gateway latency; end-to-end time to first token. | Compare similar workloads at relevant percentiles. Separate gateway time from upstream time. |
| Cost visibility | Attributed usage; discrepancies against upstream billing records. | Distinguish estimates, missing usage, and reconciled costs. |
| Product value | Integration effort saved or cost per successful application task. | Task success needs application-level evidence, not just gateway logs. |

Set targets from customer needs and a measured baseline. Avoid universal thresholds. Collect only necessary telemetry. Do not log sensitive prompts or responses by default.

### Questions to resolve with real cases

- Which repeated problem is painful enough for teams to adopt a shared gateway?
- Which capabilities and data policies must survive any routing or provider change?
- Which content risks need shared guardrails? Which need application-specific checks?
- Who defines each policy, and where must it be enforced?
- How should limited capacity be shared across teams?
- Who operates the gateway, handles incidents, and approves policy changes?
- What evidence would show that the extra layer costs more than it helps?

**Reusable lesson:** start with a shared problem and clear ownership. Add capabilities that solve that problem. Do not turn the gateway into a home for every AI feature.

[Back to the AI gateway module](README.md)
