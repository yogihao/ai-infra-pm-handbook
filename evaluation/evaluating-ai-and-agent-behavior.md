# Evaluating AI and Agent Behavior: Outputs, Decisions, and Actions

**Evaluate whether intelligence produces a supported result and takes appropriate actions, not whether its answer sounds convincing.**

These methods apply whether the intelligence runs inside an MCP server or in an external agent. MCP even supports optional server-requested generation through [client sampling](https://modelcontextprotocol.io/specification/2025-11-25/client/sampling). Deployment location does not determine the evaluation method.

Use the [evaluation foundation](evaluation-from-development-to-production.md) for release gates and continuous production evaluation. The expense-review examples below are illustrative.

## 1. Identify the behavior being evaluated

Not every model-powered operation is an agent.

| Behavior | Main evaluation question |
| --- | --- |
| Extract receipt fields | Are the values correct, and are missing values handled honestly? |
| Retrieve company policy | Is the evidence relevant, applicable, and from the correct policy version? |
| Generate a review brief | Are important facts covered and claims supported by evidence? |
| Investigate across multiple steps | Does the agent choose appropriate tools, recover from problems, and stop safely? |

A multi-step agent needs output evaluation and action evaluation. A correct final answer does not excuse an unauthorized action taken along the way.

Start with the individual behavior to locate failures. Then evaluate the integrated task with the intended tools and business environment.

## 2. Define success for one business situation

Suppose the task is:

> Prepare an expense review brief and identify evidence the approver still needs.

| Case element | Example |
| --- | --- |
| Initial state | The expense is pending; one receipt is unavailable. |
| Authority | The user may review the request but may not access banking fields. |
| Expected output | Correct permitted facts, source references, and an explicit evidence gap. |
| Expected behavior | Gather relevant evidence and stop without approving or rejecting the request. |
| Forbidden behavior | Invent receipt contents, reveal restricted data, or claim that approval occurred. |

Describe acceptable outcomes rather than requiring exact wording or one fixed tool sequence. Several sequences may be valid. Require a particular step only when the business contract demands it.

Include ordinary tasks, ambiguous requests, missing evidence, changing state, and instructions embedded in retrieved documents. The agent should treat document content as evidence, not as authority to change its task.

## 3. Inspect outputs, actions, and effects

| Evidence | What to check |
| --- | --- |
| Final output | Factual accuracy, evidence coverage, source support, and honest limitations. |
| Observable tool calls | Appropriate tools and arguments, necessary clarification or confirmation, and bounded retries. |
| Business state | Required effects occurred and prohibited effects did not. |
| Execution record | Completion, latency, model usage, upstream calls, and cost. |

Do not judge only the final text. An agent may claim success without completing an action, or may complete an action and then misreport it.

Separate agent behavior from server enforcement. If the agent attempts a forbidden action and the server blocks it, the access control worked, but the agent still made an unsafe choice. Record both findings.

Permission and business-rule enforcement must not depend on the agent's judgment. The [MCP server evaluation](evaluating-mcp-servers.md) checks those controls directly.

## 4. Match the judge to the claim

Use exact checks for amounts, IDs, structured fields, and business-state changes. Use a written human rubric for explanation quality and usefulness.

A model judge can assist with qualitative review, but calibrate it against human-reviewed cases. It is not the authority on permissions or transaction correctness. The handbook's [evaluation-set guidance](evaluating-model-apis.md#4-build-a-representative-evaluation-set) covers judge calibration and held-out examples.

For the review brief, a simple rubric could ask:

- Are the required facts correct and supported by the cited evidence?
- Are important gaps or conflicting evidence made explicit?
- Does the brief stay within the task and avoid unsupported recommendations?

A source link alone is not enough; the source must support the claim.

## 5. Compare complete outcomes, not the best run

Repeat cases where model behavior varies. Reset the business state between trials so one run does not change the next run's task.

Compare candidates against a baseline using the same cases and declared conditions. Record the model, prompt, retrieval configuration, tool versions, and operating limits. Keep some cases held out from tuning.

Report task success, critical violations, unnecessary refusals, completion time, and cost separately. Show important case groups so easy tasks do not hide failures on missing evidence or restricted actions.

Measure cost per successful task using the cost of all attempts, including failures and retries. If no tasks succeed, that metric is undefined, not zero.

Use the foundation's [production feedback loop](evaluation-from-development-to-production.md#close-the-production-feedback-loop) to add reviewed real-world failures to the evaluation set. Do not treat every user correction as a verified answer.

**Reusable lesson:** evaluate what the intelligence says, what it does, and what actually happens. Keep those findings connected to the complete capability's release decision.

[Back to the evaluation foundation](evaluation-from-development-to-production.md) | [Back to the evaluation module](README.md)
