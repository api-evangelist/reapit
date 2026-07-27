---
name: Turn an enquiry into a registered applicant and a booked viewing
description: >-
  The core UK agency flow — a portal or website lead arrives, becomes a contact,
  is registered as an applicant with requirements, and is put in a negotiator's
  diary as a viewing or valuation appointment.
api: graphql/reapit-foundations.graphql
surfaces: [graphql, rest, mcp]
operations: [GetEnquiries, GetEnquiryById, CreateContact, CreateApplicant, CreateApplicantRelationship, GetConfigurationsByType, GetNegotiators, GetOffices, GetProperties, CreateAppointment, GetAppointments, CreateJournalEntry]
mcp_tools: [contacts_create, applicants_create, appointments_create, appointments_get_types, appointments_search, negotiators_search, offices_search, properties_search]
generated: '2026-07-26'
method: generated
---

# Turn an enquiry into a registered applicant and a booked viewing

This is the flow every UK estate agency runs dozens of times a day, and the one
most PropTech integrations need first. In Reapit's model a **Contact** is the
person; **Applicant** is the *role* that person plays when they are looking to
buy or rent. Do not conflate them.

Read `skills/reapit-authenticate-and-address-a-customer.md` first — every step
below assumes `Authorization`, `api-version: 2020-01-31` and `reapit-customer`
are set.

## 1. Read the lead

Enquiries are what the CRM shows as *Internet Registrations*: leads awaiting a
negotiator's review before conversion.

```graphql
query { GetEnquiries(pageSize: 25, pageNumber: 1) { _embedded { id } } }
```

`GetEnquiryById` fetches one. `enquiryType` accepts multiple values
(`salesApplicant`, `lettingsApplicant`, `salesProperty`, `lettingsProperty`).

If the lead originates in *your* system rather than Reapit's, skip to step 2 and
use `CreateEnquiry` only if you want it to land in the agent's review queue.

## 2. Create the person

```
CreateContact
```

Send only what you have. `marketingConsent` is first-class and matters — setting
it to deny fires the `contacts.optedout` webhook, which downstream marketing
systems watch.

## 3. Register the applicant and link it to the contact

```
CreateApplicant
CreateApplicantRelationship
```

`CreateApplicant` captures requirements — `buying` or `renting` price bands,
`internalArea` / `externalArea`, `departmentId`, `officeIds`, `negotiatorIds`,
`source`. `CreateApplicantRelationship` is the join: it takes `applicantId` plus
a polymorphic `associatedType` / `associatedId` pair, so an applicant can be
linked to a **Contact** or a **Company**.

Resolve the reference data first rather than guessing ids:

```
GetConfigurationsByType(type: appointmentTypes)   # also sellingReasons, taskTypes, ...
GetNegotiators / GetOffices / GetAreas
```

## 4. Find matching stock

```
GetProperties(pageSize: 25, pageNumber: 1, embed: [images, offices])
```

`GetProperties` filters on `age`, `agentRole`, `landlordId`, `lettingStatus`,
`locality`, `parking`, `sellingStatus`, `situation`, `style` and more. Paging is
mandatory: `pageSize` defaults to 25 and maxes at 100.

Watch the boolean trap — **omitting** a boolean filter applies no filter at all;
it is not the same as passing `false`.

## 5. Book the appointment

```
CreateAppointment
```

Needs `typeId` (from `GetConfigurationsByType(type: appointmentTypes)` — `VL` is
valuation), `start`, `end`, `propertyId`, `organiserId`, `negotiatorIds`,
`officeIds`, and the attendee. All date-times are **ISO 8601 UTC with a
time-zone designator**; date-only fields reject a time component.

Verify with:

```graphql
query { GetAppointments(start: "2026-08-01T00:00:00Z", end: "2026-08-08T00:00:00Z",
                        embed: [negotiators, offices, property]) { _embedded { id } } }
```

`start` and `end` are **required** (`String!`) on `GetAppointments` — a diary
query is always a window.

## 6. Leave a trail

`CreateJournalEntry` writes a timestamped event into the CRM activity feed
against the property or contact, so the negotiator sees what your integration
did. Agents notice when an integration is silent.

## Conventions that bite

- **Creates return 201 with an empty body.** The new resource's URL is in the
  `Location` header; re-`GET` it if you need the record.
- **Updates are PATCH, return 204 empty, and require `If-Match`** carrying the
  `_eTag` you read, quotes included. A stale eTag returns **412** — re-fetch and
  replay.
- **There is no Idempotency-Key.** A retried `POST` after a timeout creates a
  duplicate applicant. De-duplicate client-side by searching first.
- **422** carries `errors[]` with `field` + `message` — read it, do not retry
  blindly.
- Rate limits: 20 req/s, 5 concurrent per customer, 250,000/day; no
  `RateLimit-*` headers, so track budget yourself.

## Same flow over MCP

`contacts_create` → `applicants_create` → `appointments_get_types` →
`appointments_create`. The MCP server embeds related entities automatically, so
appointments come back with negotiators, office and property attached.

## References

- `conventions/reapit-conventions.yml`
- `errors/reapit-problem-types.yml`
- `data-model/reapit-data-model.yml`
- `mcp/reapit-tool-crosswalk.yml`
