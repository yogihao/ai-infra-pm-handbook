# How to Evaluate a Model API: From Benchmarks to Product Fit

This is a practical evaluation framework, not a ranking of providers. Examples and thresholds are illustrative, not measured results.

## 1. Problem: which API is good?

One API has a strong public benchmark score. Another has lower token prices. A third starts streaming sooner.

None of these facts alone tells a PM which API to choose.

**A good model API meets a particular workload's quality and service requirements at an acceptable cost.** An interactive assistant and an overnight document-processing job can reasonably choose different APIs.

Use the [metrics primer](model-api-metrics.md) for definitions. This article focuses on the decision.

## 2. Evaluate the model and the service

Model quality and serving quality are related, but distinct.

The model must produce useful results. The API must deliver them with the required reliability, speed, capacity, and operational constraints.

Even endpoints offering the same named model can differ in serving configuration, limits, latency, and pricing. Model names alone do not prove identical behavior. Evaluate the exact endpoint and version you plan to use.

| Dimension | Evidence to collect | Misleading shortcut |
| --- | --- | --- |
| Task quality | Correctness, usefulness, and required schema or tool-call accuracy on representative tasks. | Choose by a public leaderboard alone. |
| Responsiveness | TTFT and completion latency at relevant percentiles and load. | Compare average latency only. |
| Reliability | Complete responses, timeouts, interrupted streams, throttling, and retry outcomes. | Count HTTP 200 as task success. |
| Capacity | Sustainable throughput while quality and latency remain acceptable. | Treat an advertised quota as achieved throughput. |
| Economics | Cost per successful task, including retries and actual usage. | Compare price per token alone. |
| Product constraints | Required features, data handling, regions, and support commitments. | Assume API compatibility means full interchangeability. |

Keep safety and policy requirements explicit where relevant. A refusal can be correct behavior on a prohibited request and an unwanted failure on a legitimate one. Evaluate both against a defined rubric.

## 3. Define the workload before choosing a winner

Write a short workload description. Include who uses it, what they need, and what counts as success.

For invoice extraction, success might require correct required fields and a valid schema. For a support assistant, it might require a grounded answer that resolves the question and follows escalation rules.

Record the expected input sizes, output sizes, languages, modalities, tool use, streaming behavior, traffic pattern, and client region.

Then separate **hard requirements** from **preferences**. A mandatory data destination or minimum quality level is not something a faster API can compensate for.

An illustrative acceptance rule could be:

> On the representative extraction set, at least 95% of cases must pass the field-accuracy rubric. At the expected peak load, p95 completion latency must stay below 8 seconds, and at least 99% of attempted tasks must complete within the deadline.

These are not recommended universal targets. Real thresholds must come from customer needs and the consequences of failure. Report sample sizes and uncertainty before treating a measured pass as conclusive.

## 4. Build a representative evaluation set

Use sanitized or permitted examples from the intended workload. Include ordinary cases, difficult cases, and relevant failure or safety cases. Keep their results visible separately so easy examples do not hide important weaknesses.

Define the rubric before comparing candidates:

- Use deterministic checks for requirements such as valid JSON, required fields, and exact values where appropriate.
- Use a written human rubric for qualities such as usefulness, grounding, and completeness.
- If using a model as a judge, calibrate it against human judgments. Do not assume its scores are unbiased or authoritative.

Schema validity alone does not establish factual correctness. A well-formed tool call can still choose the wrong tool or arguments.

Keep a held-out set for the final comparison. Repeatedly adjusting prompts against every example can make a weak candidate look stronger than it is on unseen work. Repeat stochastic cases where variation matters.

## 5. Compare under declared conditions

For an initial comparison, keep the task, prompts, tool definitions, output requirements, and output bounds comparable. Record model versions and generation settings. Identical parameter names or values do not guarantee identical behavior across models.

Use separate rounds for a common baseline and candidate-specific tuning. Record the tuning effort so an extensively optimized candidate is not presented as an out-of-the-box result.

Measure from the client location that matters to users. Record test dates, cache conditions, retries, and any gateway routing or fallback. A fallback can improve the application's outcome while hiding failure of the original endpoint.

Start at expected traffic, then test a controlled range of higher loads if authorized. Measure whether useful throughput rises, queues grow, latency worsens, or throttling increases. Limit test cost and avoid disrupting shared capacity.

Report both first-attempt and final outcomes after retries. A fast retry-assisted result is useful, but it does not mean the first attempt was reliable.

An API scorecard should keep these denominators distinct:

| Measure | What it answers |
| --- | --- |
| Completed responses divided by attempted calls | Did the API deliver a complete response? |
| Rubric-passing tasks divided by attempted tasks | Did the user get an acceptable result? |
| Total measured cost divided by successful tasks | What did each acceptable outcome cost? |

State how deadlines, cancellations, and missing usage are handled. Do not silently drop failures or assign missing costs a value of zero. If no tasks succeed, cost per successful task is undefined, not zero.

## 6. Make the tradeoff explicit

Suppose two candidates pass the same mandatory requirements:

| Illustrative observation | Candidate A | Candidate B |
| --- | --- | --- |
| Interactive experience | Faster first visible output. | Slower first visible output. |
| Accepted results | Both meet the required quality threshold. | Both meet the required quality threshold. |
| Cost per successful task | Higher. | Lower. |

For an interactive assistant, A may justify its cost. For a batch job that only needs completion before a deadline, B may be preferable.

Do not reduce hard requirements to a weighted score that allows a disqualifying failure to be averaged away. Eliminate non-compliant candidates first. Then compare the remaining quality, cost, and service tradeoffs.

If evidence is insufficient, record the uncertainty and the next required measurement. A framework does not create an outcome before testing.

## 7. Re-evaluate the deployed product

A direct provider test and an application test answer different questions. First isolate endpoint behavior. Then evaluate the real path with gateway policies, routing, retries, and application processing enabled.

The gateway can observe request outcomes, usage, and upstream behavior. It usually cannot establish whether a response solved the user's task without application evidence. Link those records with request or task identifiers.

Repeat a bounded evaluation when model versions, prompts, routing, prices, or workload change. Monitor drift in quality and service behavior rather than treating the initial selection as permanent.

## 8. PM decision record

1. **What is the workload and acceptance rule?** Name the user outcome and the hard requirements.
2. **What evidence supports the choice?** Record evaluation-set version, endpoint version, dates, settings, load, and uncertainty.
3. **Why this tradeoff?** Explain which remaining quality, latency, reliability, and cost differences matter.
4. **When should the decision change?** Define re-evaluation triggers and ownership.

**Reusable lesson:** select for successful user outcomes under realistic conditions, not for the largest benchmark score or the lowest headline price.

[Back to the metrics primer](model-api-metrics.md) | [Back to the AI gateway module](README.md)
