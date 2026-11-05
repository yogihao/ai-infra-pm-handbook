# Who Should Own an AI Gateway API Key?

**Share project access and accounting, not a single secret.** Developers can have separate personal credentials while their requests contribute to the same project's usage. Applications should have identities that do not depend on an employee.

This is a product design recommendation, not a report of a deployed system. The example is illustrative. The scope is credentials that clients present to the gateway, not the upstream provider credentials the gateway uses.

## 1. Problem: personal keys or one shared project key?

An organization has several developers working on one application. The team wants to track model usage by project.

Personal API keys seem to fragment usage. A shared project key seems simpler, but when someone leaves, they may still have a copy. Rotating it can disrupt every application using it.

This is not a choice between accountability and project accounting. It is a reason to separate identity, credentials, and usage attribution.

## 2. One project can have many credentials

An API key authenticates a caller. It does not have to be the unit that owns the bill.

| Dimension | Question it answers | Example |
| --- | --- | --- |
| Organization | Which tenant does this request belong to? | Acme |
| Project | Which work receives the usage and cost? | Search |
| Principal | Which user or application identity is calling? | Alice or Search production |
| Credential | Which key authenticated the request? | Alice's development key |
| Creator | Who issued the credential? | Alice or an authorized administrator |

A **principal** is a user or a nonhuman identity, such as a service account. The creator is an audit field, not necessarily the principal that owns the credential.

For example, Alice's key, Bob's key, and a production service-account key can all be scoped to the Search project. Usage rolls up to Search, with breakdowns by principal and credential. Each key remains independently revocable.

**For an initial product, scope each key to one project.** Creating another key should not create another project allowance. See [rate-limit scopes](rate-limit-scopes.md) for shared allowances and optional per-key caps.

This distinction also appears in [OpenAI's API key and service-account guidance](https://developers.openai.com/api/docs/guides/api-keys): it recommends user-owned project keys for personal development and service accounts for applications with independent permissions and lifecycles. That is a provider example, not a requirement to copy its entire identity model.

## 3. Make personal keys self-service and application keys independent

User ownership does not mean an administrator must manually create and distribute everyone's keys.

| Use case | Recommended platform behavior |
| --- | --- |
| Developer using the API directly | A member creates a personal key for an authorized project, if organization policy permits. Ownership is bound to the signed-in user. |
| Deployed application or automated job | An authorized member creates a project service account and its credentials. Separate applications and environments where independent permissions or rotation are needed. |
| Credential administration | Authorized admins manage permissions, expiry, and revocation through their own logins. Managing a key need not reveal its existing secret. |

For application keys, provide the secret to the server-side workload through controlled secret storage. Do not distribute it to every project member or embed it in browser code. [OpenAI's credential guidance](https://developers.openai.com/api/docs/guides/api-keys) describes showing the full service-account key only at creation and storing it in a secret manager.

Showing a secret once reduces exposure. It does not prove that the creator never copied it.

## 4. Set the rules in the platform; check requests at the gateway

**The platform stores who can do what. The gateway checks each API request against those rules.** The management side is often called the *control plane*.

Suppose Alice creates a development key for the Search project:

| Step | What happens |
| --- | --- |
| Set access | An admin grants Alice access to Search and sets allowed models and limits. Alice creates a key within that access. |
| Send a request | Alice's application sends the key with a model request. |
| Identify the caller | The gateway validates the key. Platform records tell it the key belongs to Alice, Search, and the Acme organization. |
| Check the rules | The gateway checks that the key is active, Alice still has access, the key permits the requested action, and applicable limits allow the request. |
| Forward and record | If allowed, the gateway forwards the request and attributes usage to Search, with Alice and the key identified in usage records. It does not log the secret. |

The application cannot charge another project simply by sending a different project name. The gateway uses the key's authorized project, not an unchecked claim from the caller.

If Alice loses access, her key must stop working for Search. A gateway may temporarily cache access rules, so define how quickly changes reach every instance. Once cached rules are too old, reject requests with a clear error if current access cannot be checked.

For a service-account key, the same flow identifies the application instead of Alice. It does not identify the human using that application.

## 5. Offboarding exposes the limit of ownership

Most raw API keys act as bearer credentials: possession is sufficient to use them within their permissions. [RFC 6750](https://www.rfc-editor.org/rfc/rfc6750.txt) defines this property for OAuth bearer tokens; it is not an API-key specification, but explains the same possession-based risk.

**Binding a key to Alice does not prove Alice made the request.** If Bob uses her copied key, the gateway attributes the request to her credential.

| When Alice leaves | Required behavior in this design |
| --- | --- |
| Her personal keys | Remove their access to the organization, including through cached authorization. Disabling console login alone is insufficient. |
| Service accounts she created | Keep the application identity active, with management assigned to remaining authorized members. Remove Alice's management and secret access. |
| Application secrets she could have copied | Rotate them to invalidate her copies. Changing ownership or removing secret-manager access does not erase an existing copy. |

There is no per-person revocation for people presenting an identical bearer secret. Separate credentials solve that problem for individual callers; service accounts solve application continuity, not secret leakage.

Support replacement keys and a bounded overlap for planned rotation. Revoke the old key after the application switches. Suspected compromise may require immediate revocation even at the cost of availability.

## 6. PM decisions

1. **Who can issue credentials?** Allow personal-key self-service within project access and organization policy. Restrict service-account administration to authorized roles.
2. **What survives an employee's departure?** Application identities should survive; personal organization access should not.
3. **How quickly does access removal take effect?** Specify and verify the revocation delay across gateway instances.
4. **Must production store a long-lived key?** Keep keys for compatibility, but support short-lived workload credentials where feasible.

Workload identity is a separate credential-delivery choice, not a replacement for project ownership. [OpenAI's guidance](https://developers.openai.com/api/docs/guides/api-keys) describes federation for supported identity providers without a long-lived OpenAI key. Short-lived bearer tokens can still leak; their shorter validity limits the exposure window.

**Reusable lesson:** bind credentials to users or applications, attribute usage to projects, and treat secret exposure separately from ownership.

[Back to the AI gateway module](README.md)
