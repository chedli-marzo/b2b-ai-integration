## User Story 1: Map HubSpot → QuickBooks Customer Workflow

### User Story
**As an** Integration Consultant,  
**I want to** map the end-to-end customer creation workflow from HubSpot to QuickBooks,  
**So that** we establish a clear, agreed-upon blueprint before writing code, preventing scope creep and architectural rework.

### Acceptance Criteria
* [x] A visual sequence diagram exists showing the exact flow from HubSpot trigger to QuickBooks confirmation.
* [x] The decision between Webhook-driven (event-based) vs. Polling (schedule-based) is explicitly documented with justification.
* [x] Edge cases (e.g., customer creation, customer update, customer merge in HubSpot) are explicitly mapped.
* [x] The business owner has reviewed and signed off on the workflow diagram. (Simulated sign-off, 2026-09-24: see [us-01-integration-design.md](us-01-integration-design.md).)

### Deliverables
1. Mermaid.js or Excalidraw sequence diagram of the integration flow.
2. A 1-page "Integration Design Document" (IDD) outlining the chosen architecture.

### Git
* **Branch:** `story/p1-us-01-hubspot-quickbooks-flow`, from `sprint/p1-s01-business-architecture`
* **Suggested commits** (us-01 was written before the repo existed, so these commit its current files):
  1. `docs(p1-us-01): add user story and knowledge notes` (`00-index.md`, `us-01-knowledge.md`)
  2. `docs(p1-us-01): add v1 diagram and review findings` (`us-01-diagram-v1.png`, `us-01-finding.md`)
  3. `docs(p1-us-01): add integration design and edge cases` (`us-01-integration-design.md`)

---

### Knowledge
Technical, business, interview and client notes: [us-01-knowledge.md](us-01-knowledge.md)

---

### Practice Task
Create a Mermaid.js sequence diagram showing: HubSpot User $\rightarrow$ HubSpot Webhook $\rightarrow$ Next.js API Route (Signature Check) $\rightarrow$ Inngest Queue $\rightarrow$ Data Transformation $\rightarrow$ QuickBooks API $\rightarrow$ Prisma Audit Log. Export this as a PNG and put it in your portfolio README.

---

### As Built

* **Deliverables:** [us-01-integration-design.md](us-01-integration-design.md) (webhook vs polling, decisions, key design details, 20 edge cases, diagram v3 with v1 kept for comparison), [us-01-finding.md](us-01-finding.md) (plain-language review of v1 for the business owner), [us-01-diagram-v1.png](us-01-diagram-v1.png) (v1, re-rendered from its Mermaid source). No code: this is a design story.
* **Decisions:**
  * Webhooks plus a nightly reconciliation job, over polling alone: speed and API quota. The nightly job covers outages longer than HubSpot's 24-hour retry window (built in Sprint 04).
  * Trigger on `lifecyclestage = customer`, over deal stage "Closed Won": a standard property, whereas deal pipelines differ between companies.
  * Merges flagged for Finance, over making the extra QuickBooks customer inactive automatically: QuickBooks owns financial history.
  * Merges detected from the `contact.merge` event, over reading the `hs_merged_object_ids` property: that property stays on the contact forever, so every later update would look like a new merge.
  * Idempotency key = hash of `subscriptionId + objectId + eventId + occurredAt`, over `eventId` alone (not guaranteed unique) and over anything including `attemptNumber` (changes on every retry).
  * Fetch the latest contact, over trusting the webhook payload: payloads carry ids only, and fetching makes out-of-order events harmless.
  * One job per contact at a time, over unlimited parallel jobs: parallel jobs for a new contact would both create it.
  * Read and update the QuickBooks customer in the same Inngest step, so a retry never reuses a stale `SyncToken`.
  * Inngest assumed as the job runner throughout the design; the formal Inngest vs Trigger.dev choice stays in Sprint 02.
* **Deviation from plan:**
  * The plan pictured a straight chain (webhook → queue → transform → QuickBooks → audit log), and the Practice Task asks for exactly that. The real flow needs branches (skip, create, update, merge), a fetch-latest step, a mapping lookup and failure paths. Drawing the chain as specified is what produced v1's errors.
  * Sign-off was simulated with `/drill biz us-01`, since there is no real client yet. Dana's two conditions (a fixed first-step price, maintenance terms) are recorded in the design doc.
  * us-02's knowledge file named the merge event `contact.merged`; the real name is `contact.merge` (fixed).
* **War story:** the first diagram had the webhook handler wait for Inngest, QuickBooks and the audit log before answering HubSpot. It looked tidy on paper. The review found that HubSpot times out after 5 seconds and retries up to 10 times, so any slow QuickBooks call would have produced duplicate customers in the finance system. Fix: store the event, hand it to Inngest, return 200 immediately, and make the rest idempotent. Result: v3 survives HubSpot's retries without duplicates. Lesson: acknowledge first, work later, and assume every message arrives more than once.
  * **Also caught:** writing the edge-case table exposed a race (two events for the same new contact running in parallel both create it), fixed with per-contact concurrency. Notes from another source claimed HubSpot has no `contact.merge` event and gave a wrong signature formula; checking HubSpot's official docs caught both. Implementing that formula would have rejected every request with a 401.
* **Numbers:** 9 findings in the v1 review (4 high priority); 20 edge cases mapped; HubSpot limits verified against its docs: 5-second timeout, up to 10 retries over 24 hours, up to 100 events per request; 5 client objections practised, 3 gaps identified.
