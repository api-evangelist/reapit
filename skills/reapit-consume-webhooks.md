---
name: Subscribe to Reapit webhooks and verify every payload
description: >-
  Stand up an endpoint, subscribe to the right topics for the scopes you hold,
  verify the Ed25519 signature, and handle Reapit's retry, ordering and
  duplicate semantics correctly. Reapit fires on changes made by ANY application
  touching the customer's CRM, which is what makes this the right integration
  substrate instead of polling.
api: asyncapi/reapit-foundations-webhooks.yml
surfaces: [rest]
operations: []
generated: '2026-07-26'
method: generated
---

# Subscribe to Reapit webhooks and verify every payload

55 topics across 17 entity domains, per-customer subscription (including the
`SBOX` sandbox), JSON-Schema event filters, asymmetric signing, and a documented
retry ladder. Events fire on changes from **AgencyCloud, Property Cloud or any
other AppMarket app** — not just from yours. That is the whole point: it is the
only way to see what the agent actually did in the CRM.

## 1. Stand up the endpoint

It must be a publicly reachable **https** URI that accepts `POST` with an
`application/json` body.

**Do not use an AWS API Gateway `execute-api` DNS name** — Reapit's internal
routing rejects it. Attach a custom domain instead.

Respond **fast** (a `202 Accepted`) and process asynchronously. Reapit treats any
**4xx** from you as a misconfiguration and will make **no retry at all**.

## 2. Subscribe

Create webhooks in the portal (`https://developers.reapit.cloud/webhooks/new`) or
programmatically through the REST API. Subscriptions are **per customer**, so a
multi-tenant app subscribes once per agency — or points every customer at one
endpoint and switches on `customerId`.

You are only offered topics your app holds the scope for. `applicants.created`
needs `applicants.read`; `properties.selling.*` needs `properties.read`. The one
asymmetry worth memorising: **`tenancies.modified` requires `tenancies.write`**,
unlike every other `.modified` topic in the table.

`application.install` and `application.uninstall` need **no scope** — subscribe
to them first so you learn the moment a customer onboards or leaves.

## 3. Narrow with an event filter (optional)

Filters are **JSON Schema** documents validated against the event; only passing
events are delivered. A filter binds to a *single* topic, so targeting
appointment `typeId` `VL` (valuation) across created *and* modified means
configuring it twice.

## 4. Verify the signature — always

Reapit signs with **Ed25519**, per app (not per webhook).

```
X-Signature: s:keyId:timestamp:signature
```

1. Split on `:` → `keyId` (2), `timestamp` (3), `signature` (4).
2. Fetch the public key: `GET https://platform.reapit.cloud/webhooks/signing/{keyId}`
   with a bearer token (an app can only read its own keys), or copy it from the
   portal's *Public Key* option. It comes back as a JWK set with `crv: Ed25519`
   and a base64url `x`.
3. Base64url-decode `x`.
4. Build the message as `timestamp + raw request body`, **with no delimiter**.
5. Verify segment 4 over that message with Ed25519.

Read the **raw** body. A JSON pre-processor or `JSON.stringify` round-trip
changes the bytes and the signature will not verify. Reapit publishes working
Node (`node-forge`) and .NET (`BouncyCastle`) examples.

A legacy symmetric scheme still exists and is documented as obsolescent — use
asymmetric.

Optionally also allowlist the three static eu-west-2 egress IPs:
`18.133.192.77`, `18.133.96.95`, `18.132.113.124`.

## 5. Read the envelope

| Field | Use it for |
|---|---|
| `eventId` | de-duplication key |
| `entityId` | the record — **not globally unique**, pair with `customerId` |
| `customerId` | which agency; `webhook-test` means it was a Ping |
| `eventTime` | ordering — delivery order is **not** guaranteed |
| `topicId` | the topic |
| `new` | new state; `null` if deleted/archived |
| `old` | prior state; `null` on `.created` |
| `diff` | field-level changes; populated only on `.modified` |
| `SendAttempts` | how many tries it took |

`new` / `old` / `diff` use the **same schema as the matching REST endpoint** —
a `contacts.modified` payload is a `GET /contacts` shape. Types ship as
`@reapit/foundations-ts-definitions`.

Your app's `metadata` is **not** included in webhook payloads.

## 6. Handle delivery reality

- **At least once.** Duplicates are possible — de-duplicate on `eventId`.
- **Unordered.** Sort by `eventTime`, never by arrival.
- **Retries:** up to 6 attempts at ≥60s, 120s, 300s, 600s, 900s.
- **No retry** for sandbox events, or when you returned 4xx.
- Webhook traffic counts toward your analytics and billing like any API call.

## 7. Test before you go live

Register the webhook **inactive**, then use the portal **Ping** function. Ping
payloads always carry `customerId: webhook-test` and
`entityId: 9e7e4181-6210-49ea-abf5-d5ce16d23647` — branch on those so test
deliveries never touch production state. Then exercise it against `SBOX`.

## The other direction

To push *your* events into the Reapit CRM for a named user, `POST /notifications`
with an envelope whose `targets.negotiatorId` lists one or more negotiators.
Needs `negotiators.read`, and display inside a live customer CRM must be enabled
by Reapit support at that customer's request.

## References

- `asyncapi/reapit-foundations-webhooks.yml`
- `scopes/reapit-scopes.yml`
- `sandbox/reapit-sandbox.yml`
