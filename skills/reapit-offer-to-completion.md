---
name: Take a sales offer through conveyancing and the chain
description: >-
  Submit an offer against a property, track it to acceptance, and follow the
  conveyancing record through the UK sale chain — the transaction spine of an
  estate agency, and the closest thing UK agency software has to an MLS
  transaction record.
api: graphql/reapit-foundations.graphql
surfaces: [graphql, rest]
operations: [GetProperties, GetPropertyById, GetOffers, GetOfferById, CreateOffer, UpdateOffer, GetConveyancing, GetConveyancingById, GetConveyancingChain, UpdateConveyancing, CreateUpwardLinkModel, DeleteUpwardLinkModel, CreateDownwardLinkModel, DeleteDownwardLinkModel, GetCompanies, CreateJournalEntry]
mcp_tools: [properties_search, properties_get_by_id, offers_search]
generated: '2026-07-26'
method: generated
---

# Take a sales offer through conveyancing and the chain

There is no MLS in the UK, so the CRM *is* the transaction record. Reapit models
the sale as three linked entities: **Property → Offer → Conveyancing**, with
conveyancing records linked upward and downward to form the chain.

## 1. Establish the property and the parties

```
GetPropertyById            # embed: [vendor, negotiator, offices, images]
GetProperties              # filter on sellingStatus to find live stock
```

`PropertyModel` is the widest entity in the schema (48 fields): address,
`selling`, `letting`, `epc`, internal/external area, rooms, keys. The selling
side is a **Vendor** (`Vendor.propertyId`); the buying side arrives as an
**Applicant**.

## 2. Submit the offer

```
CreateOffer
```

`OfferModel` carries `applicantId`, `propertyId`, `amount`, `date`, `status`,
plus `inclusions`, `exclusions` and `conditions` — the negotiated terms, not just
the number. Read back with `GetOffers` / `GetOfferById`, embedding
`[applicant, property, negotiator, conveyancing]`.

## 3. Move the offer's status

```
UpdateOffer
```

`PATCH` semantics: send only the fields you are changing, include the `_eTag` in
`If-Match`, expect **204 No Content**. Status transitions fire webhooks that the
rest of the agency's stack listens for — `offers.accepted`, `offers.rejected`,
`offers.withdrawn`.

## 4. Follow the conveyancing record

An accepted offer produces a **Conveyancing** record — sales progression. It is
the second-widest entity in the schema (47 fields) and it is where the whole
professional cast is wired together:

```
GetConveyancing / GetConveyancingById   # embed: [property, offer, vendor,
                                        #         vendorSolicitor, buyerSolicitor]
UpdateConveyancing
```

Company references on the record: `vendorSolicitorId`, `buyerSolicitorId`,
`mortgageLenderId`, `mortgageBrokerId`, `mortgageSurveyorId`,
`additionalSurveyorId`, `externalAgentId` — each resolves through
`GetCompanies` / `GetCompanyById`.

## 5. Work the chain

```
GetConveyancingChain
CreateUpwardLinkModel     /  DeleteUpwardLinkModel
CreateDownwardLinkModel   /  DeleteDownwardLinkModel
```

`upwardChainId` and `downwardChainId` are self-references from one conveyancing
record to another. `GetConveyancingChain` walks them. This is the artefact UK
agents actually manage — a stalled link three properties up is what kills a sale,
and it is visible here and nowhere else.

## 6. Watch it happen instead of polling

Subscribe rather than poll. The property status events are precise:

| Topic | Meaning |
|---|---|
| `properties.selling.instructed` | status became forSale / forSaleUnavailable |
| `properties.selling.askingpricechanged` | asking price moved |
| `properties.selling.underoffer` | underOffer / underOfferUnavailable |
| `properties.selling.exchanged` | contracts exchanged |
| `properties.selling.completed` | sale completed |
| `properties.selling.withdrawn` | withdrawn |
| `properties.selling.lostinstruction` | instruction lost to another agent |
| `conveyancing.modified` | any change to a sales-progression record |

All require `properties.read` / `conveyancing.read`. See
`skills/reapit-consume-webhooks.md`.

## Conventions that bite

- Offers and conveyancing records are **per-customer** ids; key them with
  `customerId + id`.
- `UpdateConveyancing` without a matching `If-Match` eTag returns **412** —
  re-fetch, replay, retry. Two negotiators editing the same chain link is exactly
  the lost-update case this prevents.
- **No idempotency key.** A retried `CreateOffer` creates a second offer. Search
  before you post.
- Financial amounts are plain numbers; there is no currency object — everything
  is the agency's local currency.

## References

- `data-model/reapit-data-model.yml` (Sales lifecycle)
- `asyncapi/reapit-foundations-webhooks.yml`
- `conventions/reapit-conventions.yml`
