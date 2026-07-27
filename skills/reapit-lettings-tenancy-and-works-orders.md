---
name: Run a lettings tenancy and raise works orders against it
description: >-
  The property-management half of the platform — create a tenancy against a
  property and applicant, run tenancy checks, and raise, chase and complete
  maintenance works orders. This is where Reapit has shipped the most change
  through 2026.
api: graphql/reapit-foundations.graphql
surfaces: [graphql, rest]
operations: [GetProperties, GetPropertyById, GetLandlords, GetLandlordById, GetApplicants, CreateTenancy, GetTenancies, GetTenancyById, GetTenancyRelationships, GetTenancyChecks, GetTenancyCheckById, CreateTenancyCheck, UpdateTenancyCheck, DeleteTenancyCheck, GetWorksOrders, GetWorksOrdersById, CreateWorksOrder, UpdateWorksOrder, GetWorksOrderItems, GetWorksOrderItemById, CreateWorksOrderItem, UpdateWorksOrderItem, DeleteWorksOrderItem, GetCompanies, GetConfigurationsByType]
mcp_tools: [tenancies_search, tenancies_get_by_id, landlords_search, landlords_get_by_id, properties_search, companies_search]
generated: '2026-07-26'
method: generated
---

# Run a lettings tenancy and raise works orders against it

Sales gets the headlines; lettings is where the recurring revenue and the
compliance risk live. Reapit's changelog through 2026 is dominated by tenancies
and works orders, which tells you where the market is competing.

## 1. Assemble the parties

```
GetPropertyById            # embed: [landlord, negotiator, offices]
GetLandlords / GetLandlordById
GetApplicants              # the prospective tenant, in their applicant role
GetConfigurationsByType(type: tenancyTypes)
```

A **Landlord** is a contact or company letting one or more properties
(`solicitorId`, `officeId`, related contacts through `LandlordRelationship`).
The tenant arrives as an **Applicant** and is bound to the tenancy through
`GetTenancyRelationships`, the same polymorphic
`associatedType` / `associatedId` join used elsewhere.

## 2. Create the tenancy

```
CreateTenancy
```

`TenancyModel` carries `typeId`, `propertyId`, `applicantId`, `negotiatorId`,
`source`, meter readings and `daysInArrears`.

Two 2026 changes matter: **`endDate` validation was removed**, so periodic
tenancies can be created without an end date; and meter readings plus last-read
dates can now be updated on *finished* tenancies (gas, electricity, water).

## 3. Run the tenancy checks

```
GetTenancyChecks / GetTenancyCheckById
CreateTenancyCheck
UpdateTenancyCheck
DeleteTenancyCheck
```

Tenancy checks are the pre-move-in and in-tenancy compliance list. They are one
of the few entities Foundations lets you **delete** (a soft delete) as well as
create and update.

Sub-resources reachable on the REST platform for the same tenancy include
`rentReviews` (which now support metadata), `extensions` (letting and management
fee type, amount, frequency), `noticeManagement` (landlord and tenant notice
dates and reasons), break clauses, allowances and responsibilities.

## 4. Raise a works order

```
CreateWorksOrder
CreateWorksOrderItem
GetWorksOrders / GetWorksOrdersById
GetWorksOrderItems / GetWorksOrderItemById
```

A works order is the maintenance job; **works order items** are its line items.
The contractor is a **Company** (`GetCompanies`, typed as supplier).

## 5. Chase and complete it

```
UpdateWorksOrder
UpdateWorksOrderItem
DeleteWorksOrderItem
```

The status ladder is `raised` → `raisedToChase` → `complete`, with `cancelled` as
the exit. On the **`raisedToChase`** status:

- On **create**, `raisedToChaseDays` may be supplied; omit it and the chase
  interval defaults to **1 day**.
- On **update**, you must supply **both** the status *and* the chase-days
  interval — sending the status alone is rejected.

## 6. Subscribe instead of polling

| Topic | Scope | Fires when |
|---|---|---|
| `tenancies.created` | `tenancies.read` | new tenancy |
| `tenancies.modified` | **`tenancies.write`** | tenancy changed |
| `worksorders.raised` | `worksorders.read` | job raised |
| `worksorders.modified` | `worksorders.read` | job changed |
| `worksorders.complete` | `worksorders.read` | job completed |
| `worksorders.cancelled` | `worksorders.read` | job cancelled |
| `contacts.landlorddetails.updated` | `contacts.read` | landlord MTD consent changed |

`tenancies.modified` is the one `.modified` topic in the whole catalog that
demands a **write** scope — request `tenancies.write` at app registration or the
topic will not be offered to you.

## Conventions that bite

- Every `UpdateTenancy` / `UpdateWorksOrder` needs `If-Match` with the current
  `_eTag`, quotes included; a mismatch is **412**, so re-fetch and replay.
- Creates return **201 empty** with a `Location` header; updates return **204
  empty**. Re-`GET` to see state.
- Certificates and invoices are named in the permissions glossary but are **not**
  in the GraphQL schema — they are REST-only. Post EPC certificates via
  `POST /properties/{id}/certificates`.
- File uploads (inspection reports, invoices) over 6 MB need a pre-signed URL:
  `POST /documents/signedUrl` → `PUT` the binary → `POST /documents` with
  `fileUrl`. URLs expire in **15 minutes**, max 10 at a time, 30 MB ceiling.

## References

- `data-model/reapit-data-model.yml` (Lettings lifecycle)
- `changelog/reapit-changelog.yml`
- `conventions/reapit-conventions.yml`
