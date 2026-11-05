# AI Gateway

How can we make model access reliable, controlled, and cost-transparent?

This module covers the layer between applications and model services. Models may be externally hosted or self-hosted. Each topic connects a technical mechanism to a product decision. Tool/MCP gateways are outside the initial scope.

## Start here

[AI Gateway: Problems, Users, and Product Boundaries](problems-users-and-product-boundaries.md)

## Interface design

[One Interface, Many Models: What Should an AI Gateway Standardize?](one-interface-many-models.md)

How a PM can decide what to unify and what to keep provider-specific.

## Identity and access

[Who Should Own an AI Gateway API Key?](api-key-ownership.md)

Separate user and application identities from project usage. Design self-service credentials and offboarding without relying on one shared secret.

## Metrics and evaluation

[Model API Metrics: Traffic, Limits, and Latency](model-api-metrics.md)

Understand tokens, RPM, TPM, concurrency, TTFT, generation speed, and measurement boundaries.

[How to Evaluate a Model API: From Benchmarks to Product Fit](evaluating-model-apis.md)

Compare task quality, service behavior, capacity, and cost against the needs of a real workload.

## Cost optimization

[Cheaper Model Calls, More Expensive Tasks?](cheaper-model-calls-more-expensive-tasks.md)

Separate caching, compression, and model routing from their hidden costs. Compare cost per successful task rather than token spend alone.

[Model Routing: Who Should Choose, and How Do We Know It Works?](model-routing-decisions-and-evaluation.md)

Separate task-aware model selection from endpoint routing. Decide what belongs in the application or gateway, and evaluate the complete workflow.

## Traffic controls

[AI Gateway Rate Limits: What Should Each Limit Control?](rate-limit-scopes.md)

Separate upstream capacity from user allowances, and choose model and API-key caps deliberately.

## Quick concept links

| Topic | Jump to |
| --- | --- |
| Traffic and capacity | [Tokens](model-api-metrics.md#2-tokens-are-units-of-model-input-and-output) · [RPM](model-api-metrics.md#3-traffic-allowances-and-concurrency) · [TPM](model-api-metrics.md#3-traffic-allowances-and-concurrency) · [Concurrency](model-api-metrics.md#3-traffic-allowances-and-concurrency) · [Throughput](model-api-metrics.md#3-traffic-allowances-and-concurrency) |
| Response speed | [TTFT](model-api-metrics.md#4-response-speed-has-several-parts) · [Inter-token latency](model-api-metrics.md#4-response-speed-has-several-parts) · [End-to-end latency](model-api-metrics.md#4-response-speed-has-several-parts) |
| Reading results | [Average](model-api-metrics.md#6-look-beyond-averages) · [p50](model-api-metrics.md#6-look-beyond-averages) · [p95](model-api-metrics.md#6-look-beyond-averages) |
| Rate-limit scopes | [Upstream quotas](rate-limit-scopes.md#3-learn-upstream-limits-do-not-assume-them) · [User limits](rate-limit-scopes.md#5-give-each-user-a-shared-allowance) · [Per-model caps](rate-limit-scopes.md#5-give-each-user-a-shared-allowance) |

[Back to the handbook](../README.md)
