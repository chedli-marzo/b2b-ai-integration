# Sprint 01: Business & Architecture

**Branch:** `sprint/p1-s01-business-architecture`, from `main`. Each story gets its own branch from this one (see the story's Git section), merged back by pull request. This branch merges into `main` when the definition of done is met.

## Sprint goal

Agree on the plan before any code is written. By the end of this sprint, the business owner and the developers share one clear, signed-off answer to four questions:

1. **What happens:** how a new customer travels from HubSpot to QuickBooks, step by step.
2. **What gets copied:** which customer details move across, how they are reformatted, and which system "owns" each detail.
3. **What happens when it breaks:** what the system does when HubSpot, QuickBooks or the data itself has a problem, and who gets told.
4. **What success looks like:** measurable targets, e.g. "99% of new customers appear in QuickBooks within 60 seconds, zero duplicates".

This sprint delivers **documents, not software**. That is deliberate: a mistake found on paper costs minutes; the same mistake found in production costs weeks and leaves messy financial data behind.

| Story | What it produces | Status |
|---|---|---|
| [us-01](us-01-hubspot-quickbooks-flow/00-index.md): HubSpot → QuickBooks flow | Flow diagram + 1-page integration design | Done: [us-01-integration-design.md](us-01-hubspot-quickbooks-flow/us-01-integration-design.md), signed off (simulated) 2026-09-24 |
| [us-02](us-02-data-mapping-sot/00-index.md): data mapping & source of truth | Field mapping table + database and validation drafts | Not started |
| [us-03](us-03-failure-modes-kpis/00-index.md): failure modes & KPIs | "What if it breaks" plan + success targets | Not started |

**Out of scope for this sprint:** application code and project setup (Sprint 02), syncing invoices (only customers are synced in Phase 1), syncing from QuickBooks back to HubSpot.

---

## For the business owner

### What you get
1. **A flow diagram:** one picture showing every step from "a salesperson marks a contact as a customer in HubSpot" to "the customer is ready for billing in QuickBooks".
2. **A translation table:** every customer detail that gets copied (name, email, address, tax ID…), where it comes from, where it goes, and how it is reformatted.
3. **A "what if it breaks" plan:** for each thing that can go wrong, what the system does automatically and who gets alerted.
4. **Success targets:** the numbers we will use to prove the integration works.

### Decisions we need from you
| Question | Why it matters |
|---|---|
| At what exact moment does a HubSpot contact become "ready for QuickBooks"? | This is the trigger. Too early and leads pollute your accounting; too late and invoices are delayed. |
| Should existing customers be copied too, or only new ones from now on? | Copying history is a separate one-off job with its own cost. |
| If a customer's email changes in HubSpot, should QuickBooks be updated, or is the billing email locked? | Decides which system is "the boss" for each detail. |
| What naming convention do you use for customers in QuickBooks? | Avoids "Acme Inc" vs "Acme Incorporated" duplicates. |
| When two duplicate contacts are merged in HubSpot, what should happen to the extra customer in QuickBooks? | QuickBooks can't merge customers automatically; we need your rule. |
| If a sync fails, who should be notified, and how? | Someone must own fixing it. |
| What is the longest acceptable delay between HubSpot and QuickBooks? | Sets the speed target. |

### Done when
**Flow (us-01)**
- [x] One picture shows, step by step, how a new customer travels from HubSpot to QuickBooks.
- [x] We chose "HubSpot notifies us instantly" over "we check HubSpot every few minutes", and wrote down why.
- [x] Special cases are covered: new customer, customer details changed, duplicate contacts merged in HubSpot.
- [x] You reviewed the picture and approved it. (Simulated, 2026-09-24.)

**Data (us-02)**
- [ ] A table lists every customer detail that gets copied and how it is reformatted.
- [ ] For each detail, one system is named the owner (e.g. HubSpot owns contact details, QuickBooks owns the billing address).
- [ ] The system remembers which HubSpot contact matches which QuickBooks customer, so it never creates the same customer twice.
- [ ] Bad data (e.g. a missing email) is stopped and flagged, never sent to QuickBooks.
- [ ] Sales Ops and Finance both approved the table.

**Reliability (us-03)**
- [ ] The "what if it breaks" plan covers: HubSpot unavailable, QuickBooks unavailable, bad data, the same update arriving several times.
- [ ] The plan says how often the system retries and what happens when it gives up: who is alerted and how the problem gets fixed.
- [ ] Success targets are agreed and written down.

---

## For the junior developer

### Read first
You can't design this without these concepts. Look each one up; you should be able to explain it in two sentences.

| Concept | One-line meaning |
|---|---|
| Webhook | HubSpot sends an HTTP POST to our app when something changes, instead of us asking repeatedly (polling). |
| Signature verification | Proves the POST really came from HubSpot (HubSpot v3 signature + timestamp). Unsigned or stale requests get a 401. |
| At-least-once delivery | HubSpot may send the same event more than once. Our system must be safe to run twice. |
| Idempotency | Running the same operation twice has the same result as running it once: no duplicate customers. |
| Durable execution (Inngest) | Background jobs that survive crashes, retry automatically with backoff, and remember which steps already succeeded. |
| Source of truth | For each field, the one system whose value wins in a conflict. |
| Exponential backoff | Retry after 1s, 2s, 4s, 8s… so we don't hammer a struggling API. |
| Dead-letter queue (DLQ) | Where jobs go after all retries fail, so they are visible and can be replayed, never silently lost. Here: a Postgres table / status. |
| Correlation ID | One ID attached to every log line of one sync, so a failure can be traced end to end. |
| FMEA | Failure Mode and Effects Analysis: a table of "what can fail → effect → how we detect it → what the system does". |

### Tasks

All deliverables go in the story's folder as markdown. Code (Prisma, Zod) is written as **drafts inside the markdown** for now; it moves into the Next.js app in Sprint 02.

**us-01 → `us-01-integration-design.md`**
- Sequence diagram as **Mermaid source** in the markdown (GitHub renders it; the PNG in [readme/](../../../readme/sequence-diagram.png) is only an export).
- 1-page design: webhook vs polling decision and why; nightly reconciliation as a fallback for missed events; an edge-case table (HubSpot event → system action).
- Diagram must show:
  - [ ] Invalid signature → 401, nothing else happens.
  - [ ] Valid → raw event stored → job enqueued → **200 returned immediately**.
  - [ ] Async part: fetch contact → validate → look up mapping → create / update / merge / skip.
  - [ ] QuickBooks returns the new customer Id and it is saved to the mapping table.
  - [ ] Retries exhausted → event marked failed (visible for Replay).

**us-02 → `us-02-data-mapping.md`**
- Mapping table with columns: HubSpot field | QuickBooks field | Transformation | Required? | Source of truth.
- Prisma model drafts: `IntegrationMapping` (HubSpot id ↔ QuickBooks id) and `SyncEvent` (every received event and its status).
- Zod schema draft for the QuickBooks customer payload.
- [ ] Every required QuickBooks field has a source or a default.
- [ ] The database itself prevents duplicates: unique constraints on the HubSpot id and on the QuickBooks id.
- [ ] The Zod schema rejects a payload with a missing required field (show one failing example).

**us-03 → `us-03-fmea.md`**
- FMEA table with columns: Failure | Effect | Detection | System behaviour | Retry? | Alert | Recovery.
- Minimum scenarios: QuickBooks 400 (bad data), 429 (rate limit), 503 (down); HubSpot down when fetching a contact; same webhook received 3 times in 1 second; invalid signature; QuickBooks access token expired.
- KPI list with numbers, and an observability plan (correlation ID, structured logs without personal data, alert on failed events).
- Error-flow diagram: correlation ID → log → failed status (DLQ) → alert.
- [ ] 4xx errors are not retried (except 429); 5xx and 429 are retried with backoff.
- [ ] Duplicates are prevented twice: Inngest idempotency key = HubSpot eventId, plus the unique constraint from us-02.

### Pitfalls (from the review of diagram v1)
- **Never call QuickBooks inside the webhook request.** Store, enqueue, return 200. HubSpot times out after a few seconds and resends, which creates duplicates.
- **Arrows mean "who calls whom".** QuickBooks never writes to our database; our Inngest function calls QuickBooks, reads the response, then writes the log.
- **The webhook only carries IDs**, not the full contact. The job must fetch the current contact from HubSpot, which also fixes events arriving out of order.
- **One HubSpot POST can contain a batch of events.** Enqueue one job per event.
- **QuickBooks access tokens expire every hour.** Token refresh is part of the flow, and a failed refresh is a failure mode.

### Definition of done (whole sprint)
- [ ] All three deliverables are in this folder, and every acceptance criterion in the us-01/02/03 files is ticked.
- [ ] Owner sign-off recorded at the top of `us-01-integration-design.md` (name + date).
- [ ] Every business question above is answered, or listed as open with a person responsible.
