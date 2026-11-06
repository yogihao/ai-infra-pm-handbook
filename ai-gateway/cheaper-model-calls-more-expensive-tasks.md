# Cheaper Model Calls, More Expensive Tasks?

A gateway vendor promises lower costs through caching, prompt compression, and model routing, with unchanged output.

The savings can be real. But these techniques make different tradeoffs. **A lower API bill is not necessarily a lower cost of getting useful work done.**

This article separates the mechanisms from the product decision. The numerical example is illustrative, not a benchmark.

## 1. What actually gets cheaper?

| Technique | Source of savings | What needs checking |
| --- | --- | --- |
| Provider prompt caching | Reuse computation for matching prompt prefixes at the provider's cache rates. | Eligibility, cache hits, read and write charges, and retention. |
| Response caching | Return a stored answer instead of making another model call. | Whether the answer is still valid for this user, data, and task. |
| Prompt pruning or compression | Send less context to the main model. | Processing overhead and whether important information survives. |
| Model routing | Send suitable requests to a cheaper model. | Router overhead and whether the selected model can do the task. |

[OpenAI's prompt-caching documentation](https://developers.openai.com/api/docs/guides/prompt-caching) describes reuse of model state for matching prefixes. It is not reuse of a finished answer. The full logical context remains available to the model.

Compression changes that context. [LLMLingua](https://arxiv.org/abs/2310.05736), an original prompt-compression paper, demonstrates shorter prompts with limited task-performance loss in its experiments. Its overhead analysis includes both compression and subsequent inference. That is evidence of a useful tradeoff, not a guarantee of identical output.

Compressing network traffic alone does not reduce model tokens. If the original text is restored before inference, token-based input charges do not shrink.

Model routing can also save money, but deciding where to send a request may require rules, embeddings, or another model inference. A wrong choice can lead to retries or quality loss. The separate [model-routing article](model-routing-decisions-and-evaluation.md) covers decision ownership, routing overhead, and evaluation.

## 2. Where does the hidden bill appear?

Suppose a document-processing workload costs $100 before optimization. Compression halves its input charges, but adds processing and correction work:

| Illustrative cost for the same workload | Original path | Compression path |
| --- | --- | --- |
| Main-model input | $60 | $30 |
| Main-model output | $40 | $40 |
| Compression processing | $0 | $10 |
| Additional human correction | $0 | $25 |
| Total | $100 | $105 |

The main-model bill fell 30%, but the total rose 5%. The correction cost is an assumed monetary value for extra human work, not a provider charge. This example assumes unchanged output volume and no other costs; it does not predict actual compression results or establish equal final quality.

The harder problem is detecting what was lost. Invalid JSON is easy to reject. A fluent summary that misses a contractual exception is not. A model judge adds expense and can make mistakes; the [evaluation guide](../evaluation/evaluating-model-apis.md) covers judge calibration.

Other costs include extra latency, support work, and maintaining optimization policies. Editing a previously stable prefix can also reduce prompt-cache reuse. Shorter input is not automatically cheaper than cached input.

## 3. Measure the task, not just the call

Our product recommendation is to compare **total cost per successful task under agreed quality and latency requirements**. This is a decision framework, not a metric these papers prove universally sufficient.

Count routing, compression, verification, retries, escalations, and applicable platform fees. Include failed attempts in total spend. Divide by tasks passing the acceptance rule, not by HTTP-successful calls. Account for operating effort and human rework separately where they cannot be reliably allocated.

The [API evaluation guide](../evaluation/evaluating-model-apis.md#5-compare-under-declared-conditions) explains call-versus-task accounting. Evaluating an optimization still requires the actual application path, including any downstream work it causes. Keep hard requirements and difficult cases visible: an attractive average must not hide an unacceptable failure.

Use the direct provider path with native caching enabled where appropriate as the baseline. Otherwise, a gateway may claim savings that were already available without it.

## 4. PM questions

1. **What changed?** Separate cached computation, removed context, reused answers, and different models.
2. **How is failure detected?** Identify what validation catches, what it misses, and who pays for correction.
3. **Is the saving net?** Compare representative traffic at the same acceptance criteria, including latency and all attempts.

The papers establish that optimization is possible. They do not establish that a particular customer's workload will save 40%, or that its outputs will remain unchanged.

**Reusable lesson:** a lower API bill is a saving only if the missing cost has not moved to retries, operations, or the user.

Sources checked on 2026-10-30. Paper results describe their experimental conditions, not current price guarantees.

[Back to the AI gateway module](README.md)
