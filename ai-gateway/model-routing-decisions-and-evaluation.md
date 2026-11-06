# Model Routing: Who Should Choose, and How Do We Know It Works?

A team wants to reduce model spend by sending easier work to a cheaper model. Should the agent choose, or should the gateway inspect each request and decide?

**Let the application define what the task needs. Let the gateway choose among approved endpoints.** This is a starting point, not a fixed rule.

This article covers routing decisions and evaluation. For the broader economics, see [cost optimization](cheaper-model-calls-more-expensive-tasks.md). Examples and prices are illustrative.

## 1. Separate intent, capability, and destination

Routing involves three different decisions:

| Decision | Useful context | Default owner |
| --- | --- | --- |
| Choose a workflow, tool, or specialist. | User intent and workflow state. | Application or agent. |
| Decide which models are acceptable for a step. | Quality requirements, validation, and consequences of failure. | Application or agent. |
| Choose among eligible endpoints. | Capacity, health, prices, and shared traffic. | Gateway or shared infrastructure. |

The application knows what the task needs. The gateway knows which endpoints are available. Start with this split, and keep every choice within approved access and data policies.

## 2. Does choosing a model require another call?

Not always. The overhead depends on the method:

| Method | Work before the task call |
| --- | --- |
| Rules or application metadata | Ordinary code; no additional model inference. |
| Embedding-based selection | An embedding computation plus a lookup or classifier. |
| A trained classifier | Classifier inference, which may run locally. |
| An LLM choosing the destination | Another generation call with its own input cost and latency. |

A local router still uses compute. An LLM router that reads the full input adds processing before the selected model reads it again.

Reuse known workflow context when possible. If the application already knows a step is extraction, it can apply a model policy without another inference. Moving routing into an agent only saves this overhead if it avoids that extra work.

[RouteLLM](https://arxiv.org/abs/2406.18665) measures learned routing and its overhead. Its savings depend on the models, workload, and prices.

## 3. Choose before generation, or escalate afterward?

A router chooses before generation. A **cascade** tries a model, checks the answer, and escalates if needed.

Suppose routing costs $0.02 per task, a cheap call costs $0.20, and a strong call costs $1:

| Illustrative workload: 100 tasks | Model and routing spend |
| --- | --- |
| Strong model directly | $100 |
| Cheap first; 30 tasks escalate | $2 + $20 + $30 = $52 |
| Cheap first; 80 tasks escalate | $2 + $20 + $80 = $102 |

Both cheap-first paths try the cheap model on every task. Costs are fixed for illustration, exclude verification and platform fees, and do not establish equal quality.

The key question is whether the check catches bad answers. Valid JSON can still contain wrong facts. Missed failures make savings look better than they are.

[FrugalGPT](https://arxiv.org/abs/2305.05176) studies cascades and identifies latency as another practical evaluation need.

## 4. Evaluate decisions and whole tasks

Evaluate the models, the routing policy, and the complete task:

| Level | Question | Evidence |
| --- | --- | --- |
| Model/API | What can each candidate do? | Task quality, usage, reliability, and latency. |
| Routing policy | Does it choose a suitable model without wasting money? | Unnecessary expensive calls, cheap-model failures, and routing overhead. |
| Application workflow | Does the complete system meet requirements? | Final task success, total spend, completion latency, retries, and escalations. |

Several models may solve the same task. Judge the result and cost, not whether the router picked a particular model name.

Compare candidate models and the routed system on representative held-out tasks. Repeat variable cases and report difficult cases separately.

Then measure the real workflow. In an agent, one model choice can cause extra tool calls or failure later. Count the whole task's success, cost, and latency.

[RouterBench](https://arxiv.org/abs/2403.12031) compares routing strategies, but does not replace multi-step application evaluation. The [API evaluation guide](../evaluation/evaluating-model-apis.md) covers component testing and judge calibration.

## 5. Put the split into practice

The application sends the task requirements: "Extraction; these models are acceptable; use this deadline and fallback policy." The gateway chooses an approved endpoint with available capacity.

If no endpoint qualifies, follow a defined wait-or-reject policy. Do not silently use an unapproved model.

**Reusable lesson:** routing is valuable when the decision costs less than it saves, and the complete task still meets its quality and latency requirements.

Sources checked on 2026-10-30. Design recommendations are our judgment, not benchmark findings.

[Back to cost optimization](cheaper-model-calls-more-expensive-tasks.md) | [Back to the AI gateway module](README.md)
