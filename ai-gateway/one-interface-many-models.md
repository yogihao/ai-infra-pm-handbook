# One Interface, Many Models: What Should an AI Gateway Standardize?

Status: draft

This is a product framework, not a provider compatibility guide. Examples are illustrative.

The [first article](problems-users-and-product-boundaries.md) explains why teams might add a gateway. This article focuses on one decision: **what to unify, and what to keep provider-specific.**

## 1. Start with the application's need

A team wants to extract invoice data using several models. A common interface could remove repeated integration work: send a document, request a supported schema, and receive a defined result.

Another team needs a provider's hosted search, citations, and conversation state. For that team, preserving native features may matter more than switching providers.

Neither choice is universally better. **Unify repeated work without hiding differences the application depends on.**

A common API also does not guarantee common quality. Two models can return valid results with different factual accuracy. Switching models still needs application-level evaluation.

## 2. Different API shapes are not the main obstacle

A special parameter or output layout does not automatically prevent unification. The important distinction is **representation versus behavior**.

| Difference | Example | Appropriate treatment |
| --- | --- | --- |
| Same meaning, different format | One API returns text in `text`; another uses `message.content`. | Translate both into a common result if the required information is preserved. |
| An additional capability | One API also returns structured citations. | Define an optional capability or retain a provider-specific extension. Route citation-required requests only to capable targets. |
| Different behavior or state | One API accepts a handle for a provider-managed conversation. | Keep it native or restrict support to compatible targets. Renaming the field cannot make the state portable. |

The same test applies to input parameters. A different name may be easy to map. A setting with no equivalent on another model cannot become portable just because the common API accepts it.

The reverse is also true: identical names do not guarantee identical meaning. For example, the same `temperature` value does not promise equivalent behavior across models.

## 3. What belongs in the common interface?

Start with capabilities that applications repeatedly need. The gateway can standardize shared controls as well as model-facing behavior:

| Area | What to standardize |
| --- | --- |
| Identity and access | Project identity, model permissions, and credential selection. |
| Request lifecycle | Request IDs, deadlines, trace correlation, and cancellation behavior. |
| Basic inference | Supported inputs, outputs, schema constraints, and generation controls. |
| Streaming | Defined content, tool-call, completion, and failure events. |
| Errors | Common error categories and diagnostic details. |
| Usage | A common record of provider-reported consumption. |

## 4. Common and native interfaces can coexist

A gateway does not have to choose between unifying everything and unifying nothing.

**Use a common interface** for behavior that can be preserved across selected models. It can follow an existing client protocol; a gateway does not need to invent a new API.

**Use provider-specific options** when a native setting fits the same request lifecycle. Validate it against the selected target and make any routing restriction explicit.

**Use a native route** when the application depends on a provider's own API and behavior. Preserve the details rather than forcing them into a misleading common format.

Native routes can still share access controls and request tracking. However, forwarding an unfamiliar payload does not mean the gateway can inspect its content or interpret its usage. Required policies must work on the chosen route.

## 5. How should a PM decide?

Ask three questions:

1. **Is it useful?** Does it reduce repeated integration work for our users?
2. **Can it work across models?** Can we keep the behavior the application needs?
3. **Is it worth the cost?** Does the benefit justify building and maintaining it?

If all three answers are yes, it is a good candidate for the common interface. If an important feature cannot be unified, keep it provider-specific.

**Reusable lesson:** unify what helps users; preserve the differences they need.

[Back to the AI gateway module](README.md)
