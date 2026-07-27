---
name: Authenticate with Reapit Connect and address the right customer
description: >-
  Obtain a Reapit Connect token with the right grant, point it at the right
  agency (or the SBOX sandbox), and send the three headers every Foundations
  call needs. Getting this wrong is the single most common failure mode against
  Reapit — it produces 401, 403 and silent cross-tenant confusion.
api: authentication/reapit-authentication.yml
surfaces: [rest, graphql, mcp]
operations: [Ping]
generated: '2026-07-26'
method: generated
---

# Authenticate with Reapit Connect and address the right customer

Every Foundations surface — REST, GraphQL, webhooks management, notifications
and MCP — sits behind one identity service: **Reapit Connect**
(`https://connect.reapit.cloud/`, an Auth0-backed OpenID Connect tenant). Tokens
live **3600 seconds**.

## 1. Pick the grant by what you are building

| You are | Grant | Consequence |
|---|---|---|
| A web/desktop app with a signed-in agency user | `authorization_code` + PKCE (S256) | Works on REST **and** GraphQL |
| A server-side integration with no user | `client_credentials` | Works on REST and MCP, **not** GraphQL |

GraphQL requires **both** an `authorization` header carrying the **idToken** and
a `reapit-connect-token` header carrying the **accessToken**. Client-credentials
clients never receive an idToken, so they are structurally excluded from GraphQL.
If you need machine-to-machine access, plan for REST.

## 2. Get a token (client credentials)

```
POST https://connect.reapit.cloud/oauth/token
Authorization: Basic base64(client_id:client_secret)
Content-Type: application/x-www-form-urlencoded

client_id=<client id>&grant_type=client_credentials
```

Response: `{ "access_token": "...", "token_type": "Bearer", "expires_in": 3600 }`.

Never run this flow from a browser — Reapit explicitly forbids client-credentials
from client-side-only applications.

For the authorization-code flow use `@reapit/connect-session` (browser) or
`@reapit/connect-session-server` (Node) rather than hand-rolling PKCE.

## 3. Send the headers

REST (`https://platform.reapit.cloud`):

```
Authorization: Bearer <access token>
api-version: 2020-01-31          # REQUIRED — omit it and the request fails
reapit-customer: SBOX            # client-credentials only; SBOX = sandbox
```

`api-version` is not optional. `reapit-customer` is only needed for
client-credentials callers — user-context tokens carry the customer code inside
the token.

MCP (`https://foundations-mcp.iaas.reapit.cloud/mcp`, Streamable HTTP): same
`Authorization` and `reapit-customer`, plus the token's `scope` **must** include
`agencyCloud/mcp.access` or the server returns **403**. Optional `x-timezone`
(default UTC) formats date-times in the text stream.

## 4. Verify the connection

On GraphQL, run the argument-free health check before anything else:

```graphql
query { Ping }
```

A `String!` back means credentials, headers and tenant are all good. On REST,
any paged GET is the equivalent smoke test.

## 5. Reach real customer data

Sandbox is self-serve. Production is not. The path is:

1. Build and prove against `SBOX`.
2. Submit the app at `https://marketplace.reapit.cloud/developer/submit-app`.
3. Pass Reapit's **app listing review**.
4. A customer administrator **installs** the app and grants the permissions it
   requested.
5. Use that customer's id in `reapit-customer`.

Uninstalling revokes access immediately. Adding a permission later requires
contacting `partners@reapit.com`, cutting an app revision, and **every already-
installed customer individually accepting** the new permission.

## 6. The identifier trap

Resource ids (`OXF18000001`, `OXF190347`) are unique **only within one
customer's database**. Store `customerId + id` as a composite key, or you will
collide the moment your app serves a second agency.

## Failure modes

| Status | Meaning | Fix |
|---|---|---|
| 401 | Missing/expired/invalid token | Refresh; tokens last 3600s |
| 403 | Valid token, insufficient scope | Request the permission via a revision; for MCP, add `agencyCloud/mcp.access` |
| 400 | Client-credentials token without `reapit-customer` | Add the tenant header |
| 502 | (MCP) upstream identity verification failure | Retry |

## References

- `authentication/reapit-authentication.yml`
- `scopes/reapit-scopes.yml`
- `sandbox/reapit-sandbox.yml`
- `conventions/reapit-conventions.yml`
