---
name: Create and syndicate property listings out of the CRM
description: >-
  Create and maintain a property record with images and marketing data, then keep
  it in sync outward. In a market with no MLS, the agent's CRM is where the
  machine-readable property record actually lives — this is the flow that feeds
  the portals and any downstream consumer.
api: graphql/reapit-foundations.graphql
surfaces: [graphql, rest, mcp]
operations: [GetProperties, GetPropertyById, CreateProperty, UpdateProperty, GetPropertyImages, GetPropertyImageById, CreatePropertyImage, UpdatePropertyImage, DeletePropertyImage, GetAreas, GetAreaById, GetDepartments, GetOffices, GetNegotiators, GetVendors, GetLandlords, GetConfigurationsByType]
mcp_tools: [properties_search, properties_get_by_id, areas_search, offices_search, negotiators_search, get_departments, get_property_attribute_options]
generated: '2026-07-26'
method: generated
---

# Create and syndicate property listings out of the CRM

The UK has no MLS. Listings are created and maintained in the agent's CRM and
pushed outward to the portals. That makes `PropertyModel` — 48 fields, the widest
entity in the schema — the canonical property record, and this flow the one most
portal, website and data integrations need.

## 1. Resolve the reference data first

Property fields are id-typed; resolve before you write.

```
GetDepartments             # sales / lettings — also gates valid attribute options
GetOffices
GetNegotiators
GetAreas / GetAreaById     # agent-defined geographies (coords or localities)
GetConfigurationsByType    # sellingReasons, keyTypes, documentTypes, ...
```

Over MCP the equivalent is `get_departments`,
`get_property_attribute_options`, `offices_search`, `negotiators_search`,
`areas_search`.

## 2. Create the property

```
CreateProperty
```

The shape divides cleanly:

- **Identity / routing:** `alternateId`, `areaId`, `departmentId`,
  `negotiatorId`, `officeIds`
- **Address:** `address` (building name/number, line1-4, postcode, countryId)
- **Sales side:** `selling` (price, qualifier, status, `sellingReasonId`,
  exchange/completion dates)
- **Lettings side:** `letting` (rent, frequency, availability, furnishing,
  `lettingStatus`)
- **Physical:** `internalArea`, `externalArea`, `rooms`, `epc`, bedrooms,
  bathrooms, receptions, `style`, `situation`, `parking`, `age`
- **Marketing:** description, summary, long description

You get **201 Created with an empty body** and the new record's URL in the
`Location` header.

## 3. Attach the imagery

```
CreatePropertyImage
GetPropertyImages / GetPropertyImageById
UpdatePropertyImage
DeletePropertyImage
```

Images carry `propertyId`, `type` (photograph, EPC, floorplan, map), `order` and
`caption` — `order` is what drives the portal carousel, so set it deliberately.

Inline uploads are capped at **6 MB total request size**. For 6-30 MB:

```
POST /propertyImages/signedUrl   { "amount": 1 }     # up to 10 URLs, expire in 15 min
PUT  <the returned URL>          <binary body>
POST /propertyImages             { ..., "fileUrl": "<the URL, query string stripped>" }
```

Use `fileUrl`, not base64 `fileData`, for the second post.

## 4. Attach the selling/letting party

A sales property has a **Vendor** (`GetVendors`, `Vendor.propertyId`); a lettings
property has a **Landlord** (`GetLandlords`, reachable via
`GetPropertyById(embed: [landlord])`). Both link back to contacts or companies
through their relationship join entities.

## 5. Read stock back for syndication

```graphql
query {
  GetProperties(pageSize: 100, pageNumber: 1, sortBy: "-modified",
                embed: [images, offices, negotiator, area, department]) {
    _embedded { id }
    totalCount
    totalPageCount
  }
}
```

- `pageSize` defaults to **25**, maximum **100**. Paging is mandatory on
  top-level collections.
- Use `embed` to collapse round trips — but note **embed sub-requests still count
  against your rate limit and billing** exactly as if you had issued them.
- Filter with `sellingStatus` / `lettingStatus` / `agentRole` / `locality` /
  `style` / `situation` / `age` / `parking`.
- **Omitting a boolean filter applies no filter** — it is not `false`.

## 6. Keep it in sync — do not re-crawl

Poll only for the initial backfill. After that, subscribe:

| Topic | Fires when |
|---|---|
| `properties.created` / `properties.modified` | any property change |
| `propertyimages.created` / `propertyimages.modified` | imagery change |
| `properties.selling.instructed` | went live for sale |
| `properties.selling.askingpricechanged` | price moved |
| `properties.selling.underoffer` / `.exchanged` / `.completed` | progression |
| `properties.selling.withdrawn` / `.lostinstruction` | came off market |

All need `properties.read`. The `.modified` payload carries a field-level `diff`,
so you can push a delta rather than a whole record.

## 7. Updates

```
UpdateProperty
```

`PATCH` — send only changed fields, set `If-Match` to the resource's `_eTag`
(quotes included), expect **204 No Content** and an empty body. A stale eTag is
**412**; re-fetch, replay, retry. This is what stops a syndication job silently
overwriting a negotiator's edit.

## Caveats worth stating plainly

- Property ids (`OXF190347`) are unique **only within one agency's database**.
  Key on `customerId + id`.
- There is **no Universal Property Identifier**, no RESO Data Dictionary mapping
  and no RESO Web API — field names are Reapit's own and follow UK agency
  practice.
- Custom fields live in `metadata`, which is scoped to *your* app and *one*
  customer, is filterable via the `metadata` query parameter
  (`$eq/$ne/$gt/$lt/$gte/$lte/$in/$nin/$con`), can be validated with a JSON
  Schema registered on `/metadataSchema` — and is **absent from GraphQL** and
  from webhook payloads.

## References

- `data-model/reapit-data-model.yml`
- `conventions/reapit-conventions.yml`
- `asyncapi/reapit-foundations-webhooks.yml`
- `rate-limits/reapit-rate-limits.yml`
