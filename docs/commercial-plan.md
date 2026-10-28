# Commercial plan

**Status:** Draft 0.1, 27 October 2026. Not a business approval, price list, or forecast.

This plan describes who the product is for, how it can be packaged, and how prices should be learned. It does not set prices. No customer interviews or willingness-to-pay studies are recorded in this repository.

## Problem

CCTV installation companies and end customers already record video. People do not watch most of it. A product can watch authorized streams for configured events, especially a person entering a restricted area, and present those events for a human to accept or reject.

The value fails if alerts are too noisy, real entries are missed and the customer believed they were covered, cameras quietly go offline, or the system is unsafe to deploy on a customer network.

## Customer segments

| Segment | Likely buyer | Likely user | Why they might buy | Primary risk |
| --- | --- | --- | --- | --- |
| CCTV installation companies | Owner or sales lead | Technician plus the end customer's guard | A paid add-on to camera projects and a reason to keep a maintenance contract | They will compare it with free NVR analytics and will not tolerate a fragile install |
| Security service companies | Operations manager | Control-room operator | Fewer screens watched continuously | Missed events become a service failure |
| Warehouses and factories | Site or security manager | Guard and supervisor | After-hours entry into defined areas | Dust, lighting, camera placement, and informal work patterns |
| Offices and commercial buildings | Facility or admin head | Reception or security | Lobbies, server rooms, loading doors | Employees and visitors are in the footage; notice and purpose matter |
| Retail stores | Store owner or loss-prevention lead | Store supervisor | Restricted stock rooms more than open shop floors | Crowding makes naive zone rules noisy |

Recommendation in [open-decisions.md](open-decisions.md) OD-006: first end-customer pilot is a warehouse or factory zone, reached through an installer if one is available. Retail shop-floor analytics are a poor first use case.

The repository contains no evidence that any of these segments have been interviewed.

## Geographic focus

Pakistan is the intended first market because that is the stated starting point, not because a market study exists here.

Practical consequences:

- Many buyers will want footage to remain on their premises. Treat customer-hosted installation as the default sale. Treat vendor-hosted ingest of live cameras as a later product.
- Installers are a credible channel because they already visit sites, pull cable, and configure NVRs.
- Power cuts and unstable links are operational requirements. Offline behavior of the local server, bounded reconnect, and visible camera loss matter more than a cloud dashboard.
- Invoices, sales tax, and contracts must follow the seller's province and business registration. This document does not determine tax.
- International expansion is a later project. A design that stores UTC timestamps, isolates tenants, and can delete a customer's data is a prerequisite, not permission to sell abroad.

## Packaging

The same software build should support the motions below. Do not fork a separate codebase per motion.

| Offer | What the customer gets | What must be true in the product | Release |
| --- | --- | --- | --- |
| Pilot | Named cameras, named users, written limitations, a fixed time period, deletion at the end | Release 1 gates | R1 |
| Customer-hosted subscription or annual license | Software on their server, camera allowance, upgrade and support terms | Entitlements, backup docs, no vendor backdoor | R1.1 for paid licensing; R1 can be a free or paid pilot if the contract says so |
| Vendor-hosted subscription | Vendor runs the server | Counsel on data roles, residency, and a location for the server. Not the first default | After OD-012 |
| Installer partner | Wholesale relationship, ability to deploy for clients, no standing access to client video unless delegated | Partner model | R1.1 |
| Installation fee | On-site configuration, zone setup, training | A checklist, not a new subsystem | Can be sold with the pilot as a service |
| Enterprise contract | Edge, hybrid, identity, uptime targets | Release 3 scope written per customer | R3 |

Camera-count and site-count limits are the clearest usage meters. Per-camera pricing is easy for installers to explain. A site fee plus a camera fee may fit organizations with many small sites. Do not choose the meter until interviews show how buyers already purchase CCTV maintenance.

### License behavior that protects the customer

Recommended commercial rule, still open as OD-017:

- Non-payment or expiry pauses live processing after the contractual grace period.
- Already stored events and evidence remain readable through retention.
- The product must not wipe a site because an invoice was late.
- Suspended tenants cannot add cameras or use paused features.
- Offline on-prem servers need a documented clock and grace behavior so a network outage is not treated as a licensing breach. That design is Release 1.1.

## What is not priced

Do not publish a rate card from this file. Costs below are categories to measure. No currency amounts are known.

### Costs to measure before a price floor

- Engineer time to install the first site, including zones and false-alert tuning.
- Support time per camera per month after the first month.
- Server specification that actually meets the pilot profile, once [ai-evaluation-plan.md](ai-evaluation-plan.md) has numbers.
- Disk for snapshots at the chosen retention.
- Model license fees, if any.
- Power, UPS, and a spare disk. These are usually the customer's costs; the quote must say so.
- Backup media and who replaces a failed disk.
- Travel to site.
- Channel margin if an installer resells.
- On-call expectation. An unpriced 24/7 promise will consume a small team.

A price floor is cost plus support plus a margin the owner chooses. A price ceiling is what buyers will pay instead of a guard's time, an NVR analytics license, or doing nothing. Both ends are unknown until measured.

## Pricing-validation plan

Do this before printing prices.

1. **Interviews.** Eight to twelve conversations with installers and end customers in the chosen segment. Ask what they watch manually, what a missed after-hours entry costs them, what they pay now for cameras and monitoring, who signs, and whether footage may leave the building. Do not lead with a feature list.
2. **Willingness to pay.** For each interview, ask for a price that is too cheap to trust, a price that is expensive but plausible, and a price that is out of the question, for a stated camera count. Record the camera count. This is a qualitative range, not a study with statistical confidence.
3. **Costed pilot.** Run two sites. Log hours of installation, tuning, and support. Log disk growth and server load from the benchmark method. Convert hours to a cost with the owner's real labor rate.
4. **Alternative comparison.** Record the competing option the buyer names, including "the guard already watches" and bundled NVR features. Do not invent competitor prices.
5. **Decision.** The owner picks a pilot price and a later list price in writing, with the meter (per camera, per site, or both) and what support hours are included. Until that note exists, sales conversations should say that pricing is not set.

No step above has been executed.

## Pilot strategy

A pilot is a paid or unpaid controlled deployment with a written end date. It is not a production certificate.

Proposed shape:

- Two pilots, sequentially if the team is small. Two cameras each unless the owner raises the count.
- Each camera has a named purpose, a named zone, and a person who is allowed to authorize processing.
- Duration on the order of four to six weeks after installation, long enough to see false alerts per camera-hour.
- Success is not "the model is accurate" in the abstract. Success is: the operator can explain each alert, the camera-down state is noticed, reconnect works on a real interruption, the buyer can state a price they would consider, and no unauthorized person accessed evidence.
- Failure that stops a sale: silent camera loss, credentials exposed, cross-customer access, or an alert rate the operator abandons.
- End of pilot: export or delete data as the agreement says, and record what happened.

The pilot agreement should state, in plain language, that people can be missed, that benign activity can alert, and that a person must decide what to do.

## Channel responsibilities

| Work | End customer | Installer, when used | Software vendor |
| --- | --- | --- | --- |
| Lawful purpose, notices, and employee or visitor disclosure | Accountable | May advise; cannot decide for the customer | Provides limitation text and retention controls. Does not provide a legal opinion |
| Cameras, power, network, RTSP reachability | Owns the site | Usually performs the work | Provides connection requirements after they are tested |
| Zones and rules | Approves | May draft under delegation in Release 1.1 | Provides the editor and explains failure modes |
| Watching and acting on alerts | Accountable | Optional monitoring service, if that is their business | Does not act as the guard |
| Software defects, patches, documented upgrade | Reports | First line if the contract says so | Accountable for the software |
| Backups | Owns the decision and the media on a customer-hosted server | May operate the task | Provides a procedure and a restore test method |
| Camera passwords | Owns | Handles for installation | Stores only as encrypted product secrets; never displays them back |

Release 1 support is best-effort under a pilot agreement. A contractual uptime percentage is a Release 3 conversation and would be false if offered now.

## Hosted service versus customer-hosted

| Topic | Customer-hosted pilot | Vendor-hosted service |
| --- | --- | --- |
| Who can physically reach the disks | Customer | Vendor, so access controls and staff procedures matter more |
| Footage path | Stays on the site LAN if the server is on that LAN | Must be designed. Default is still local processing, not raw RTSP over the internet |
| Tenancy | One organization in the simple case | Many organizations; isolation tests are a launch gate |
| Identity of the vendor's administrators | No vendor account by default | No standing evidence access. Time-bounded grant only |
| Data location | Customer's machine | Must be chosen. There is no assumption of a Pakistan cloud region |
| Outage | Local power and disk | Vendor's provider plus the site's uplink |

An installer who hosts several customers on one machine is in the vendor-hosted column for isolation purposes, even if the installer owns the hardware.

## International expansion

Before a sale outside Pakistan, review at least:

- Whether any person in the footage is covered by another country's privacy law.
- Where the server and backups sit.
- Contract, tax, invoicing, and support language and hours.
- Export or encryption rules that apply to the software, if any, for the destination.
- A support path that does not depend on someone driving to the site.

None of that review is done.

## Marketing language that is allowed later

Only after evidence exists:

- Describe measured behavior on a named camera and site, with the date and the limits.
- Describe features that are in the shipped build and covered by tests.

Language that is not allowed:

- "Never misses an intruder", "AI security guard", "compliant with" a named law, "military grade", or a percentage accuracy that is not in an evaluation report.
- Customer quotes that were not given.
- Implied emergency-response automation.

## Commercial risks

See the consolidated register in [implementation-plan.md](implementation-plan.md). The business risks that can kill the product even if the software works are: no installer will add another fragile box, buyers will not pay above the cost of support, a single missed event becomes a dispute, and a privacy complaint arises from cameras that were never properly authorized.
