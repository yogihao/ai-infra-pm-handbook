# Evaluation Architecture: From Development to Production

Evaluation should answer two questions: **is this change ready to release, and is the deployed system still behaving acceptably?**

CI tests answer only part of that. Production permissions, dependencies, data, and model behavior can change without a code deployment.

This article outlines a shared evaluation design for MCP tools and AI workflows. The office automation examples are illustrative, not measured results.

## 1. Separate capability contracts from intelligent behavior

MCP describes how capabilities are exposed and called. It does not determine whether their implementation uses an LLM. A server may use ordinary application logic, call a model internally, or request model generation through MCP's optional [sampling capability](https://modelcontextprotocol.io/specification/2025-11-25/client/sampling).

Consider a tool that prepares an expense review brief. It might run an internal agent to gather evidence and write the brief. An external agent might then use that brief to answer the user.

| Evaluation scope | Main question |
| --- | --- |
| Intelligence module | Does extraction, retrieval, generation, or planning work correctly with controlled inputs? |
| Complete MCP tool | Does the capability honor its contract, permissions, and business rules across the real integration? |
| End-to-end agent workflow | Does the user receive the right outcome from the combined system? |

These scopes overlap deliberately. A correct brief built from an unauthorized receipt is still a tool failure. A correct tool result misrepresented by the external agent is an end-to-end failure.

The [MCP tools specification](https://modelcontextprotocol.io/specification/2025-11-25/server/tools) defines invocation and result conventions. It does not establish that a business task succeeded.

Use focused evaluations to diagnose failures, and combined evaluations to decide whether the complete capability is ready.

## 2. Choose the dimensions that matter

Security, correctness, and cost are dimensions to check at the relevant scopes, not separate system layers.

| Dimension | Main question | Example evidence |
| --- | --- | --- |
| Security | Can data or authority be misused? | Tenant isolation and rejection of unauthorized actions. |
| Compliance | Are applicable policies and obligations followed? | Approved data destinations, retention controls, and audit records. |
| Correctness | Are results and business effects right? | Accurate facts, valid state transitions, and supported conclusions. |
| Stability | Does operation remain reliable over time? | Availability, error trends, and resource growth. |
| Robustness | What happens when conditions are abnormal? | Safe behavior under timeouts, malformed input, and dependency failure. |
| Engineering performance | How efficiently does the system operate? | Tail latency, throughput, saturation, and upstream call count. |
| Cost | What does useful operation cost? | Infrastructure, upstream, and model cost per successful task. |

Policy owners must define compliance requirements. Automated checks provide evidence; they do not independently establish legal compliance.

Do not combine every dimension into one score. Lower cost cannot compensate for unauthorized disclosure.

## 3. Build one evidence loop

Start with a small runner and version-controlled cases, not a large evaluation platform.

```text
Cases, policies, and acceptance criteria
                  |
                  v
Execution in isolated tests, staging, and production
                  |
                  v
Results, traces, and outcome reviews
                  |
                  v
Release and operational decisions
                  |
                  v
New failures become regression cases
```

Each case needs an acting identity, initial business state, input, expected outcome, and forbidden behavior. For example: an approver can read an expense summary, cannot see banking fields, and must be told when permitted evidence is unavailable.

Use deterministic assertions for permissions and state changes. Use reviewed examples and suitable quality judgments for generated explanations. Repeat variable AI cases rather than trusting one successful run.

Reports should identify the build, configuration, policy, and case-set versions, plus model and prompt versions when relevant. Distinguish product failures from evaluation setup failures. Missing credentials or an empty test run must not count as a pass.

Collect only the diagnostic data needed. Evaluation traces can contain the same sensitive information as production traffic.

## 4. Give each lifecycle stage a job

The criteria should connect development and operations, but the same tests should not run everywhere.

| Stage | Evaluation approach | Decision |
| --- | --- | --- |
| CI, before merge | Fast, isolated contract, permission, and business-rule tests. | Merge or block. |
| Staging, before release | Real integrations, deployed authentication, load, and controlled failure scenarios. | Qualify or reject the release candidate. |
| Production canary | Limited rollout with synthetic checks and comparison against the current version. | Expand, hold, or halt. |
| Continuous production | Operational metrics, audit checks, and approved sampling of task outcomes. | Continue, investigate, disable a capability, or roll back. |

Promote the artifact that was evaluated. Treat prompt, model, and policy changes as behavior changes that also need evaluation.

Keep enterprise credentials out of untrusted pull-request runs. Use isolated records and test identities before production.

Production monitoring can reveal outages and latency regressions, but healthy metrics do not prove that explanations are accurate. Outcome evaluation remains necessary.

## 5. Evaluate production without creating new harm

Use three complementary sources of evidence:

| Method | What it provides | Important limit |
| --- | --- | --- |
| Passive observation | Metrics, traces, and audit events from existing activity. | Does not establish every outcome's correctness. |
| Synthetic journeys | Known tasks using dedicated identities and controlled records. | Does not cover every real user's permissions or data. |
| Sampled outcome review | Evidence about actual user tasks. | Requires approved access, privacy, and retention controls. |

Do not replay real approvals or payments to compare versions. Shadow execution is appropriate only when data handling and side effects are controlled. Even reads may disclose data to a model or incur charges.

If a hard violation appears during a canary, halt expansion and contain the affected capability. Small traffic volume does not make an authorization defect acceptable.

Rollback also has limits. Reverting code cannot undo an approval already submitted. Define who disables the tool, reconciles uncertain operations, and handles business recovery.

### Close the production feedback loop

Collection supplies evidence; evaluation judges it against an expected outcome, rule, or target.

| Step | What to do |
| --- | --- |
| Collect | Capture necessary tool results, errors, metrics, and user corrections under privacy and retention controls. |
| Select | Keep a representative sample alongside failure-focused and high-risk cases. |
| Evaluate | Check results, permissions, and business effects against explicit criteria. |
| Act | Investigate, alert, disable a capability, or improve its implementation. |
| Preserve | Review and sanitize useful cases, then add them to the regression suite for future releases. |

User feedback is a signal, not automatically ground truth. Confirm the expected behavior before preserving a case.

Continuous evaluation does not mean checking every task immediately. Operational alerts can run near real time; expensive quality reviews can be sampled or scheduled. Authorization and business-rule enforcement must still happen during execution, not wait for a later evaluation.

## 6. Agree on gates, targets, and owners

Use **hard gates** for invariants such as permission enforcement and valid business transitions. An observed violation blocks release; passing the cases is evidence, not proof of universal safety.

Use **quality and operating targets** for task completion, accuracy, availability, latency, and cost. Set thresholds from workflow needs and a measured baseline.

A safe refusal during an outage can pass the failure-handling check while remaining an incomplete user task. Record both.

Before implementation, resolve three decisions:

- What does this capability promise, and what must never happen?
- What evidence is required before release and during production operation?
- Who can halt rollout or disable the capability, and who owns recovery?

The first milestone is a small, repeatable evaluation that informs both release and production decisions. Continue with [MCP server evaluation](evaluating-mcp-servers.md) and [AI and agent behavior](evaluating-ai-and-agent-behavior.md) for the focused evaluation methods.

**Reusable lesson:** evaluate before release, limit exposure during release, and keep checking after release. Separate evaluations to find failures; combine their evidence to govern the complete user outcome.

[Back to the evaluation module](README.md)
