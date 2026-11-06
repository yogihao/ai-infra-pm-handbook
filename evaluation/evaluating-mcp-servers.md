# Evaluating MCP Servers: Contracts, Permissions, and Business Effects

**Evaluate what a tool promises, not just whether it returns a successful response.** A valid response can still contain the wrong records, expose restricted information, or hide a failed business operation.

This article focuses on the complete capability exposed through MCP. Use the [evaluation foundation](evaluation-from-development-to-production.md) for shared evaluation dimensions, CI/CD, and the production feedback loop.

The expense-review examples below are illustrative.

## 1. Turn the tool's promise into a contract

Suppose `get_expense_review_context` retrieves information needed to review one expense request.

| Contract element | Example |
| --- | --- |
| Acting identity | An authenticated approver in the correct tenant. |
| Input | A request ID. |
| Result | Permitted request details, current state, evidence, and source references. |
| Side effects | No business-state changes. |
| Limitations | Missing permitted evidence is identified; restricted information is not disclosed. |
| Failure behavior | Retrieval failure is distinguishable from a valid empty result. |

This contract is broader than a schema. It defines what the caller can rely on.

The [MCP tools specification](https://modelcontextprotocol.io/specification/2025-11-25/server/tools) defines input schemas, optional output schemas, and result conventions. Check those, but also check the meaning of the returned data. A correctly typed amount can still belong to the wrong expense.

## 2. Test business situations, not only arguments

Each case should combine an identity, initial state, invocation, and expected outcome.

| Situation | What must hold |
| --- | --- |
| Authorized approver | The correct request and permitted fields are returned. |
| Different employee or tenant | Unauthorized access is denied without leaking restricted information. |
| Restricted attachment | Neither its contents nor restricted metadata appear in the result. |
| Revoked access | A later invocation uses current permissions, not earlier authorization. |
| Unavailable evidence | The result reports the permitted evidence gap without inventing content. |
| Upstream failure | The failure is explicit, not disguised as "nothing found." |
| Read operation | The request's business state remains unchanged. |

Test both allowed and denied access. A tool that rejects everyone does not fulfill its contract.

Inspect the tool result itself, not only the external agent's final answer. Sensitive information reaching the model is already a disclosure.

When adding writes, add cases for invalid transitions, concurrent edits, repeated submissions, and uncertain outcomes after timeouts. Verify the resulting business state. A timeout does not prove that the operation failed, so blindly retrying may duplicate it.

## 3. Exercise the real MCP interface

Use an MCP client to initialize a connection, discover the tool, and call it through the intended transport. Calling an internal function alone misses integration failures.

| Interface check | What it establishes |
| --- | --- |
| Discovery and schema | The intended client can find the capability and understand its inputs. |
| Input validation | Invalid arguments are rejected before business effects occur. |
| Result structure | Results match the declared contract and any output schema. |
| Errors | Callers can distinguish execution failure from a valid business result. |
| Pagination, where used | Results are bounded, and continuation follows the documented behavior. |

MCP distinguishes [protocol errors from tool execution errors](https://modelcontextprotocol.io/specification/2025-11-25/server/tools#error-handling). Evaluate both rather than treating every response as success.

Test deployed authentication as well as business permissions. Being authorized to connect to the server is not permission to read every OA record.

## 4. Check reliability and operating limits

Use controlled environments to test slow dependencies, rate limits, concurrency, and sustained load. Confirm that the tool meets its deadlines, bounds its work, and does not multiply upstream calls through uncontrolled retries.

Measure latency and cost alongside successful results. Include upstream charges and internal model usage when present. A fast error is not evidence that the useful operation is fast.

Use test adapters for quick, repeatable failure cases, then verify the real OA integration with isolated records. A fake permission model cannot establish that production permissions are enforced correctly.

Compliance checks should follow the organization's defined requirements: for example, permitted data destinations and attributable audit events. Do not copy sensitive tool payloads into evaluation logs by default.

## 5. Include intelligence when it is part of the promise

If the tool changes from returning evidence to producing a review brief, the brief's quality becomes part of the tool contract.

Use [AI and agent evaluation](evaluating-ai-and-agent-behavior.md) to assess that intelligence separately, then include its results in the complete tool's release decision. A good model-output score cannot excuse an authorization failure.

Before release, resolve three questions:

- Does the tool fulfill its contract for permitted users and reject prohibited operations?
- Does it behave safely under failure, concurrency, and retries?
- Can it meet the workflow's performance, cost, and audit requirements?

**Reusable lesson:** check the interface, the returned information, and the resulting business state. Protocol success is only one part of capability correctness.

[Back to the evaluation foundation](evaluation-from-development-to-production.md) | [Back to the evaluation module](README.md)
