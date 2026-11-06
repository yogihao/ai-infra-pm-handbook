# Building an MCP Server for a Complex Office Automation System: Where to Start

Start with one useful user workflow, not a catalog of endpoints. Make that workflow work end to end under the existing system's permissions and business rules.

Office automation is a useful example of a complex business system. It combines data, workflow state, permissions, and actions that affect other people. The same design problem appears in HR, finance, procurement, and customer management systems.

This article is a design framework. The approval workflow below is illustrative, not a report of a deployed system or measured results.

## 1. Why office automation is more than data access

Consider an employee expense request. Reading its amount is simple. Understanding what a particular person can do with it is not.

| Dimension | Example | Design consequence |
| --- | --- | --- |
| Data | Request, receipts, employee, cost center, and policy. | A useful answer may require several related records. |
| Workflow state | Draft, submitted, under review, or paid. | Valid actions depend on current state. |
| Permissions | An employee, manager, and finance reviewer see different information. | Access must be checked for the acting user. |
| Business rules | Spending limits and approval delegation. | Rules belong in trusted implementations, not model reasoning alone. |
| Side effects | Submission starts review; approval may trigger payment processing. | A tool invocation can have consequences beyond its immediate result. |

These dimensions interact. Being allowed to view a request does not imply permission to approve it. A manager's permission may change when a delegation expires.

The challenge is therefore not just "let an agent call the system." It is "let an agent help a user complete a task without changing the meaning of the system's rules."

## 2. Separate MCP support from business capability

The [MCP tools specification](https://modelcontextprotocol.io/specification/2025-11-25/server/tools) defines how clients discover and invoke tools, including input schemas and tool results. It provides the connection contract. It does not define your company's approval rules.

A server can speak MCP correctly and still expose the wrong capabilities: too many low-level operations, incomplete results, or actions that bypass existing controls.

Treat the work as two linked responsibilities:

| Responsibility | Main question |
| --- | --- |
| MCP integration | Can the intended client connect, discover tools, invoke them, and interpret results and errors? |
| Business capability | Can the user accomplish a useful task with correct data, permissions, and behavior? |

Do not treat every existing endpoint as a tool that must be exposed. Existing interfaces are integration inputs, not the product roadmap.

## 3. Choose one small but complete user outcome

Start with concrete user goals rather than "connect the entire OA system."

| Workflow | Example request | Initial scope |
| --- | --- | --- |
| Find information | "What is our travel reimbursement policy?" | Read-only. |
| Understand work | "Which requests are waiting for my approval?" | Read-only. |
| Prepare an action | "Draft a reimbursement from these receipts." | Draft creation. |
| Execute an action | "Submit this reimbursement." | Explicit confirmation. |
| Make a consequential decision | "Approve this purchase." | Stricter controls; potentially excluded initially. |

These are illustrative candidates, not a required rollout order. All require permission checks; confirmation alone does not authorize an action.

For this article, choose one workflow from "Understand work":

> As an approver, show me the expense requests waiting for my review and explain a selected request using information I am allowed to see.

This is more useful than "fetch a request by ID." It includes finding the right work and understanding it. It is also narrower than "automate expense approval."

Before implementation, agree on a short workflow contract:

| Decision | Example scope |
| --- | --- |
| User | An authenticated expense approver. |
| Outcome | Find pending requests and understand one selected request. |
| Included | Request summary, current state, permitted supporting evidence, and source links. |
| Excluded | Approving, rejecting, editing, and deciding whether an expense complies with policy. |
| Failure behavior | Report unavailable evidence and retrieval failures; do not invent missing details. |
| Success evidence | Users complete the task accurately with less manual navigation than today. |

Read-only is a useful starting point when it solves a real problem. It reduces mutation risk, but it does not eliminate privacy risk. Sending a restricted document to a model is already a disclosure.

Choose a workflow with observed demand, accessible integrations, and a clear completion condition. If the real need requires a write, include the required controls rather than presenting a read-only demo as a solution.

## 4. Trace the existing rules before designing tools

Follow the selected workflow through the current application. Identify where its data comes from, which implementation enforces access, and what happens when dependencies fail.

| Investigation | Decision it informs |
| --- | --- |
| How is the user identified, including tenant and delegation? | How the MCP invocation maps to an OA identity. |
| Who determines which requests are pending for that user? | Which existing workflow logic to reuse. |
| Are attachments and sensitive fields separately restricted? | What can enter the tool result and model context. |
| Can records change between listing and reading? | How to represent current state and freshness. |
| Are supported integrations complete enough for the task? | Whether the first workflow is feasible without new OA work. |

Reuse authoritative business logic where possible. Do not query tables directly merely because that is easier than reproducing the application's permission and workflow behavior.

The [MCP authorization specification](https://modelcontextprotocol.io/specification/2025-11-25/basic/authorization) describes transport-level authorization for HTTP-based connections. That is distinct from deciding whether a user may read a particular expense or receipt. Both must be addressed.

Derive the acting identity from verified authentication. Do not trust a model-supplied employee ID as proof of identity. Access to the MCP server must not silently become the full authority of an upstream service account.

## 5. Design a small tool interface around the task

The first workflow might need only two tools:

| Illustrative tool | Input | Result |
| --- | --- | --- |
| `list_my_pending_expense_reviews` | Page size and cursor. | Permitted request summaries and a next-page cursor. |
| `get_expense_review_context` | Request ID. | Permitted request details, supporting evidence, current state, and source references. |

The second tool is a deep module: it hides necessary lookups and permission checks behind a small interface. The model should not need to reconstruct joins, interpret internal department codes, or guess which attachment endpoint to call.

Keep the tool's purpose explicit. "Get review context" must not also mark a request as reviewed or approve it.

Define result behavior as carefully as input fields:

- Return stable IDs and source references so claims can be checked.
- Bound result size and make pagination explicit.
- Distinguish an empty result from a failed lookup.
- Distinguish permitted but unavailable evidence from evidence successfully retrieved.
- Do not reveal restricted fields, attachment names, or record existence through error messages.

MCP supports structured tool results and optional output schemas. Use them to make these distinctions consistent. The [tools specification's error handling section](https://modelcontextprotocol.io/specification/2025-11-25/server/tools#error-handling) also distinguishes protocol errors from tool execution errors. A failed OA lookup should not become a success-shaped empty list.

## 6. Build one end-to-end path

Implement a thin path from the intended client to the existing system:

```text
MCP client
    -> MCP connection and tool dispatch
    -> Verified identity and access enforcement
    -> Expense review module
    -> OA integration adapter
    -> Existing OA system and its business rules
```

These are responsibilities, not mandatory separate deployments. Keep protocol handling separate from business behavior, and preserve the OA system's checks.

For a shared enterprise deployment, verify the intended client's transport and authentication support early. A local connection demo does not prove that enterprise sign-in and delegated access work.

Build with realistic test identities and records. Connect invocation traces to upstream calls so failures can be investigated. Avoid logging sensitive request bodies and receipts by default.

Retrieved comments and documents are evidence, not instructions. An instruction embedded in a receipt must not change the agent's task or authority. Server-side checks remain necessary even when the agent follows the expected prompt.

Do not build a universal connector framework or expose every department's operations before this path works.

## 7. Decide whether the first workflow actually works

A successful tool response is not enough. Evaluate the complete user task with representative examples and compare it with the current manual process.

| Test case | Required evidence |
| --- | --- |
| Normal request | The user finds the right request and receives an accurate, source-backed explanation. |
| No pending work | The result clearly says there are no pending requests, rather than implying a failure. |
| Restricted record or field | The tool does not disclose information the user cannot access. |
| Missing attachment or upstream outage | The answer identifies the limitation without fabricating evidence. |
| State or permission changes | Current checks govern retrieval; the answer does not present an old list as guaranteed current. |
| Instructions embedded in content | They do not redirect the task or authorize additional access. |

Agree on task completion, accuracy, and time-saving targets before the pilot. Treat authorization failures as release blockers, not as errors that a high average success rate can offset. Passing a finite test set is evidence, not proof that every access case is safe.

If writes are later added, they need separate acceptance criteria for confirmation, state revalidation, duplicate prevention, and uncertain outcomes after timeouts. They are not just extra endpoints.

The first milestone is a working user outcome, documented tool contracts, enforced permissions, and evidence from a small pilot. Selecting the second workflow is a separate discovery decision.

For the shared evaluation design, see [Evaluation Architecture: From Development to Production](../evaluation/evaluation-from-development-to-production.md).

**Reusable lesson:** start with one user outcome and carry the existing system's rules through the entire path. MCP makes capabilities callable; useful and trustworthy workflow design makes them worth calling.

[Back to the MCP module](README.md)
