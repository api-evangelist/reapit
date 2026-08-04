# Reapit (reapit)

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

Reapit is a United Kingdom-headquartered supplier of agency CRM and property management software for estate and letting agents, best known for AgencyCloud and Property Cloud, and operating across the UK and Ireland, Australia and New Zealand, and Denmark (through Mindworking). In a market with no MLS, Reapit sits in the middle of the value chain: listings are created and maintained in the agent's CRM and pushed outward to the two dominant consumer portals, Rightmove and Zoopla, so the CRM — not any cooperative database — is where the machine-readable property record actually lives. Its API posture is unusually open for this sector and is the counter-example to "certified but unreachable": Reapit Foundations is a genuine developer platform with a public documentation site at foundations-documentation.reapit.cloud, a self-serve registration form at developers.reapit.cloud/register behind published Developer Terms and Conditions, an immediately usable SBOX sandbox, OpenID Connect authentication through Reapit Connect with a live anonymous discovery document, a REST API covering roughly thirty CRM resource domains, a public GraphQL proxy, a real webhooks system with sixty-plus event topics, and an alpha Model Context Protocol server. The honest caveat is the contract: the Swagger/OpenAPI document that powers the Interactive API Explorer is served from platform.reapit.cloud/docs and requires a bearer token, so no machine-readable spec could be harvested anonymously, and reaching any real agency's data requires the app to pass Reapit's AppMarket listing review and then be installed by that customer, who grants the scopes. There is no RESO Web API or Data Dictionary certification, no OData $metadata document and no Universal Property Identifier anywhere in Reapit's stack — RESO is a North American, NAR-driven construct and the UK has no MLS to certify against. Reapit publishes no open data; the open UK property layer belongs to HM Land Registry and Ordnance Survey, not to the CRM vendors.

**APIs.json:** [https://raw.githubusercontent.com/api-evangelist/reapit/refs/heads/main/apis.yml](https://raw.githubusercontent.com/api-evangelist/reapit/refs/heads/main/apis.yml)

## Tags

- Real Estate
- United Kingdom
- PropTech
- CRM
- Estate Agents
- Property Listings
- Property Management
- Rentals
- Conveyancing
- Australia

## Timestamps

- **Created:** 2026-07-26
- **Modified:** 2026-07-26

## APIs

### Reapit Foundations Platform REST API

The core Foundations REST API over the Reapit agency CRM data platform. It is documented as a hypermedia REST API with date-based versioning (the `api-version: 2020-01-31` header is required), optimistic concurrency via ETag/If-Match, embedded and linked resources, pre-signed URL file upload, and a metadata service for custom entities and custom fields. Resource domains named in the platform permissions glossary include applicants, areas, appointments, certificates, companies, contacts, conveyancing, documents, enquiries, identity checks, invoices, journal entries, keys, landlords, negotiators, offers, offices, properties, property alarm data, referrals, sources, tasks, telephony notifications, tenancies, transactions, vendors and works orders. Published rate limits are 20 requests per second, 5 concurrent requests per customer and 250,000 requests per day. The Interactive API Explorer at developers.reapit.cloud/swagger renders a Swagger document, but that document is served from platform.reapit.cloud/docs and requires a bearer token — it returned HTTP 200 with "Could not load Swagger documentation" to an anonymous client, so no spec was harvestable.

- **Human URL:** [https://foundations-documentation.reapit.cloud/api/api-documentation](https://foundations-documentation.reapit.cloud/api/api-documentation)
- **Base URL:** `https://platform.reapit.cloud`

#### Tags

- Real Estate
- United Kingdom
- CRM
- Property Management
- Property Listings

#### Properties

- [Documentation](https://foundations-documentation.reapit.cloud/api/api-documentation)
- [API Reference](https://developers.reapit.cloud/swagger)
- [Authentication](https://foundations-documentation.reapit.cloud/api/api-documentation#authentication)
- [Scopes](https://foundations-documentation.reapit.cloud/platform-glossary/permissions)
- [Glossary](https://foundations-documentation.reapit.cloud/platform-glossary)
- [Rate Limits](https://foundations-documentation.reapit.cloud/api/api-documentation#rate-limits)
- [Versioning](https://foundations-documentation.reapit.cloud/api/api-documentation#versioning)
- [Errors](https://foundations-documentation.reapit.cloud/api/api-documentation#errors)
- [Sandbox](https://foundations-documentation.reapit.cloud/api/api-documentation#sandbox-mode)
- [Changelog](https://foundations-documentation.reapit.cloud/whats-new)
- [Troubleshooting](https://foundations-documentation.reapit.cloud/troubleshooting/platform-api)
- [Terms of Service](https://foundations-documentation.reapit.cloud/developer-terms-and-conditions)
- [Sign Up](https://developers.reapit.cloud/register)
- [LLMs.txt](https://foundations-documentation.reapit.cloud/llms.txt)

### Reapit Foundations GraphQL API

A GraphQL proxy over the Foundations Platform REST API, released publicly out of internal beta and used in production by Reapit's own Geo Diary AppMarket app. The stated objective is a schema identical to the REST platform with no extension or deviation. Visiting the endpoint in a browser loads a GraphQL Playground. Authentication is unusual and restrictive: it requires BOTH an `authorization` header carrying the Reapit Connect idToken and a `reapit-connect-token` header carrying the accessToken, which means only the authorization code flow works — client credentials machine-to-machine clients cannot use it. Metadata and Metadata Schema are absent from the schema because arbitrary JSON cannot be strongly typed. No additional charge over the REST API; consumption is billed on the downstream REST calls the proxy makes.

- **Human URL:** [https://foundations-documentation.reapit.cloud/api/graphql](https://foundations-documentation.reapit.cloud/api/graphql)
- **Base URL:** `https://graphql.reapit.cloud/graphql`

#### Tags

- GraphQL
- Real Estate
- United Kingdom
- CRM

#### Properties

- [Documentation](https://foundations-documentation.reapit.cloud/api/graphql)
- [Authentication](https://foundations-documentation.reapit.cloud/api/reapit-connect)
- [Issue Tracker](https://github.com/reapit/foundations/issues)

### Reapit Foundations Webhooks

Real-time outbound event delivery from the Reapit CRM to an endpoint you host. Webhooks are created either in the developer portal UI at developers.reapit.cloud/webhooks/new or programmatically through the REST API, are subscribed per customer (including the SBOX sandbox), and support event filters and optional inclusion of semi-structured payload data. Published topics span applicants, appointments, companies, contacts, conveyancing, documents, enquiries, identity checks, landlords, offers, offices, properties, property images, referrals, tenancies and vendors, with created / modified / read variants plus lifecycle events such as offers.accepted, offers.rejected, offers.withdrawn, appointments.cancelled, appointments.confirmed, properties.selling, contacts.optedout, and application.install / application.uninstall for AppMarket installation notifications. Delivery uses exponential backoff on failure, payloads are signed (signing key retrievable from platform.reapit.cloud/webhooks/signing/{id}), and source IP addresses are published for allowlisting.

- **Human URL:** [https://foundations-documentation.reapit.cloud/api/webhooks](https://foundations-documentation.reapit.cloud/api/webhooks)
- **Base URL:** `https://platform.reapit.cloud`

#### Tags

- Webhooks
- Events
- Real Estate
- United Kingdom

#### Properties

- [Documentation](https://foundations-documentation.reapit.cloud/api/webhooks)
- [Webhooks](https://foundations-documentation.reapit.cloud/api/webhooks#available-topics)
- [Console](https://developers.reapit.cloud/webhooks/manage)
- [Security](https://foundations-documentation.reapit.cloud/api/webhooks#webhook-source-ip-addresses-allowlisting)

### Reapit Foundations MCP Server (Alpha)

An alpha Model Context Protocol server exposing the Foundations platform to AI agents over Streamable HTTP at a single endpoint. Authentication is a Reapit Connect JWT that must carry the `agencyCloud/mcp.access` scope (a token without it is rejected with 403), plus a `reapit-customer` tenant header for machine-to-machine clients. Registered tool domains are appointments (the richest, including open-house attendee management), properties (search, appraisals, key movements, driving directions), configuration lookups, contacts, applicants, companies, offices, negotiators, areas, offers, landlords, vendors and tenancies. Related entities are embedded automatically. Early access is by form submission; the endpoint returned HTTP 401 to an anonymous probe.

- **Human URL:** [https://foundations-documentation.reapit.cloud/api/mcp-alpha](https://foundations-documentation.reapit.cloud/api/mcp-alpha)
- **Base URL:** `https://foundations-mcp.iaas.reapit.cloud/mcp`

#### Tags

- MCP
- AI
- Agents
- Real Estate
- United Kingdom

#### Properties

- [Documentation](https://foundations-documentation.reapit.cloud/api/mcp-alpha)
- [MCP Server](https://foundations-mcp.iaas.reapit.cloud/mcp)
- [Authentication](https://foundations-documentation.reapit.cloud/api/reapit-connect)

### Reapit Connect

Reapit's hosted OpenID Connect identity service, which fronts every other Foundations API. It supports the authorization code flow for user-context applications and the client credentials flow for server-to-server integrations, issuing bearer access tokens with a one-hour lifetime. Application scopes are selected at app registration in the developer portal and are granted per customer at install time. The OIDC discovery document is served anonymously at https://connect.reapit.cloud/.well-known/openid-configuration (HTTP 200) and shows an Auth0-backed tenant with PKCE (S256), device authorization, back-channel authentication, JWKS, dynamic client registration and token revocation endpoints.

- **Human URL:** [https://foundations-documentation.reapit.cloud/api/reapit-connect](https://foundations-documentation.reapit.cloud/api/reapit-connect)
- **Base URL:** `https://connect.reapit.cloud`

#### Tags

- Authentication
- OpenID Connect
- OAuth 2.0
- Identity

#### Properties

- [Documentation](https://foundations-documentation.reapit.cloud/api/reapit-connect)
- [OpenID Connect Discovery](authentication/reapit-connect-openid-configuration.json)
- [Authentication](https://connect.reapit.cloud/.well-known/openid-configuration)
- [SDK](https://foundations-documentation.reapit.cloud/app-development/connect-session)
- [Troubleshooting](https://foundations-documentation.reapit.cloud/troubleshooting/reapit-connect)
- [Changelog](https://foundations-documentation.reapit.cloud/reapit-connect-updates)

### Reapit Foundations Notifications API

An inbound API that lets a third-party application push real-time event notifications into Reapit products for display to named CRM users. Requests POST to /notifications using an envelope-and-payload contract, where the envelope carries the notification type and a `targets` object listing one or more `negotiatorId` values; multiple targets trigger a fan-out. Callers need the `negotiators.read` scope and GET /negotiators to map their own users back to Reapit negotiators. Sandbox testing requires an active Developer Edition subscription and a dedicated negotiator ID, and display inside a live customer CRM must be enabled by Reapit support at the customer's request.

- **Human URL:** [https://foundations-documentation.reapit.cloud/api/notifications](https://foundations-documentation.reapit.cloud/api/notifications)
- **Base URL:** `https://platform.reapit.cloud`

#### Tags

- Notifications
- Events
- Real Estate
- United Kingdom

#### Properties

- [Documentation](https://foundations-documentation.reapit.cloud/api/notifications)
- [Authentication](https://foundations-documentation.reapit.cloud/api/reapit-connect)

### Reapit AgencyCloud Desktop API

A custom URI-scheme API, not an HTTP API, used by AppMarket web applications hosted inside the AgencyCloud desktop CRM to drive the desktop from the embedded app. Links prefixed `agencycloud:` are structured REST-style against AgencyCloud's primary and secondary screens — property (load, matching, journal, offers, search), applicants (load, matching, journal, search), appointments (load diary), contacts (search, load, journal), works orders (load, search) and related screens. Testing requires AgencyCloud Developer Edition, obtained from the Desktop page of the developer portal; Reapit does not provide technical support for Developer Edition, and it is documented as beta with known gaps such as desktop-added images and documents not appearing on the platform.

- **Human URL:** [https://foundations-documentation.reapit.cloud/api/desktop-api](https://foundations-documentation.reapit.cloud/api/desktop-api)

#### Tags

- Desktop
- CRM
- Real Estate
- United Kingdom

#### Properties

- [Documentation](https://foundations-documentation.reapit.cloud/api/desktop-api)
- [Developer Edition](https://developers.reapit.cloud/desktop)

## Common Properties

- [Website](https://www.reapit.com/)
- [Documentation](https://foundations-documentation.reapit.cloud/)
- [Developer Portal](https://developers.reapit.cloud/)
- [Sign Up](https://developers.reapit.cloud/register)
- [Authentication](https://foundations-documentation.reapit.cloud/api/reapit-connect)
- [Terms of Service](https://foundations-documentation.reapit.cloud/developer-terms-and-conditions)
- [Marketplace](https://marketplace.reapit.cloud/apps)
- [Partner Program](https://www.reapit.com/company/partner-program)
- [Platform](https://www.reapit.com/platform)
- [Support](https://foundations-documentation.reapit.cloud/help)
- [Status Page](https://status.reapit.com/)
- [Changelog](https://foundations-documentation.reapit.cloud/whats-new)
- [FAQ](https://foundations-documentation.reapit.cloud/faqs)
- [LLMs.txt](https://foundations-documentation.reapit.cloud/llms.txt)
- [GitHub Organization](https://github.com/reapit)
- [SDK](https://foundations-documentation.reapit.cloud/app-development/foundations-ts-defintions)
- [Code Examples](https://github.com/reapit/foundations-code-examples)
- [Blog](https://www.reapit.com/resources/blog)
- [Privacy Policy](https://www.reapit.com/legal/privacy-policy)
- [Contact](https://www.reapit.com/company/contact)
- [Linked In](https://www.linkedin.com/company/reapit)
## Access

- **Access gate:** self-serve, with a second gate behind it. Anyone can register a developer account at https://developers.reapit.cloud/register, accept the Developer Terms and Conditions, and call the `SBOX` sandbox — reads and writes — the same day. No estate-agent licence, no board or association membership, no MLS agreement, and no IDX/VOW data licence exist in this market.
- **Production data:** gated twice. The app must be submitted at https://marketplace.reapit.cloud/developer/submit-app, pass Reapit's AppMarket listing review, and then be installed by an individual agency customer who grants the requested permissions (OAuth scopes) at install time. Uninstalling revokes access immediately.
- **Auth model:** OpenID Connect over OAuth 2.0 via Reapit Connect (Auth0-backed). Authorization code flow for user-context apps; client credentials for machine-to-machine. Bearer tokens, `expires_in` 3600. Every call needs `api-version: 2020-01-31`, and machine-to-machine calls need `reapit-customer: <customer-id>` (`SBOX` for the sandbox). The OIDC discovery document is served anonymously and is saved verbatim at [authentication/reapit-connect-openid-configuration.json](authentication/reapit-connect-openid-configuration.json).
- **Scopes:** application permissions map to CRM resource domains split Read/Write across 27 domains (applicants, appointments, contacts, conveyancing, offers, properties, tenancies, works orders and more). Two scope strings appear verbatim in the docs: `agencyCloud/mcp.access` for the MCP server and `negotiators.read` for the Notifications API.
- **Machine-readable contracts:** none harvestable. The Swagger document behind the Interactive API Explorer is fetched from `https://platform.reapit.cloud/docs` with an injected bearer token; anonymously that path returns HTTP 200 with `{"errorType":"string","errorMessage":"Could not load Swagger documentation. Please try again later","trace":[]}` and every neighbouring spec path returns 401. Zero OpenAPI/Swagger files were saved.
- **RESO posture:** no RESO reference found anywhere. No Web API or Data Dictionary certification, no OData `$metadata`, no Universal Property Identifier. Reapit's own docs state that resource identifiers "are only unique to a specific customer's database and *not* globally unique" — the structural opposite of a UPI. The UK has no MLS to certify against.
- **Open data:** none published by Reapit. The open UK property layer is HM Land Registry (Price Paid and ownership data under the Open Government Licence) and Ordnance Survey, not the CRM vendors.
- **Rate limits:** 20 requests/second, 5 concurrent requests per customer, 250,000 requests/day.
- **Agent posture:** an alpha Model Context Protocol server at `https://foundations-mcp.iaas.reapit.cloud/mcp` (Streamable HTTP, ~40 tools across 13 domains), plus a published `llms.txt` index and markdown mirrors of every documentation page.

See [review.yml](review.yml) for the full probe log, RESO posture, access gate, auth model, and harvest provenance.
