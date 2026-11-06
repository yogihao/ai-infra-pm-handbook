# Evaluation Tooling: What to Reuse and What to Build

**Choose evaluation tooling by the jobs it must do, not by whether it advertises AI evaluation.** Collecting traces and generating scores are useful, but they do not establish that a business task succeeded.

Here, "tooling" means the software used to evaluate a system. It is different from the MCP tools being evaluated.

This is a capability framework, not a product ranking. The office automation examples are illustrative. Use the [evaluation foundation](evaluation-from-development-to-production.md) for the shared lifecycle and release decisions.

## 1. Identify the jobs

| Capability | What it does | Expense-review example |
| --- | --- | --- |
| Dataset management | Versions cases, context, and expected outcomes. | A pending request, its permissions, and a missing receipt. |
| Execution harness | Sets up cases, runs them, and captures results. | Authenticate as the approver and invoke the MCP server. |
| Evaluators or graders | Judge outputs, actions, and effects against criteria. | Check permitted fields, explanation accuracy, and unchanged business state. |
| Tracing and observability | Connects evidence across the execution path. | Link the user task, model calls, MCP invocations, and OA operations. |
| Human review | Supports annotation, correction, and investigation. | An expert checks whether a brief omits important evidence. |
| Reporting and release integration | Compares versions and supplies release evidence. | Surface a permission regression or increased task cost. |

These are capabilities, not six mandatory products. One platform may cover several. A trace viewer is not automatically an evaluation system, and an LLM grader is not a complete execution harness.

## 2. Match the instrument to the question

| Evaluation target | Appropriate starting point |
| --- | --- |
| MCP behavior, permissions, and business effects | Automated tests, an MCP client, and controlled business fixtures. |
| Generated answers and agent decisions | An evaluation harness with exact checks, quality rubrics, and human review. |
| Latency, throughput, and sustained reliability | Load tests and operational monitoring. |
| Production task quality | Sampled traces, evaluators, and review queues. |
| Compliance requirements | Policy-specific checks and reviewable evidence. |

For example, the official [MCP Inspector](https://github.com/modelcontextprotocol/inspector) provides interactive inspection and a CLI for automation. It can help exercise a server. It does not supply your company's expense-approval rules or the expected outcome for each case.

An AI evaluation platform does not replace ordinary software testing. Use exact assertions for permissions and state changes; use appropriate quality judgments for explanations. See the focused guides for [MCP servers](evaluating-mcp-servers.md) and [AI and agent behavior](evaluating-ai-and-agent-behavior.md).

## 3. Reuse infrastructure; own the business checks

| Usually reuse | Usually build or configure for the application |
| --- | --- |
| Test execution, timeouts, and reporting. | Test identities, records, and environment reset. |
| Dataset storage and experiment comparison. | Representative scenarios and verified expected outcomes. |
| Tracing and review interfaces. | Business-state assertions and explanation rubrics. |
| CI integrations and result export. | Acceptance thresholds and release policy. |

A platform can run a permission check, but someone must define who should have access. It can record a model score, but someone must establish whether that score is useful.

Start with the existing test runner for business correctness, an AI evaluation harness where needed, and shared tracing. Add a platform when collaboration, dataset management, or production review becomes a real bottleneck.

## 4. Try one workflow before choosing a stack

Use the same small set of cases to assess candidate tooling: a normal expense review, restricted information, and unavailable evidence.

The tooling should let the team set up each case, run it, inspect the evidence, explain the verdict, and compare it with a baseline. Include both exact business checks and a reviewed explanation-quality judgment.

Before adopting it, resolve three questions:

- Can it represent our identities, workflow state, custom checks, and multi-step actions?
- Can it protect sensitive evidence through access, redaction, retention, and approved data destinations, including any external model judge?
- Can it version and export cases and results, connect to CI and production review, and operate within our cost budget?

Budget for evaluation itself: model judges, repeated runs, trace storage, and human review. Do not send every production payload to an evaluator by default.

Use the foundation's [production feedback loop](evaluation-from-development-to-production.md#close-the-production-feedback-loop) to turn reviewed findings into reusable cases. Tooling should support that process, not replace the judgment behind it.

**Reusable lesson:** reuse the machinery for running, observing, and comparing evaluations. Keep ownership of what correct business behavior means.

[Back to the evaluation module](README.md)
