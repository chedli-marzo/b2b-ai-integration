# us-01 Knowledge: Map HubSpot → QuickBooks Customer Workflow

Story: [00-index.md](00-index.md)

---

## Technical Knowledge
* **What it is:** Defining the control flow of data between two disparate systems.
* **Why it matters:** Coding before mapping leads to "spaghetti integrations" that break when edge cases (like a user updating a name twice in 5 seconds) occur.
* **How it works:** HubSpot emits a `contact.creation` or `contact.property_change` webhook $\rightarrow$ Your Next.js API route receives it $\rightarrow$ Validates signature $\rightarrow$ Pushes to a background job queue (Inngest/Trigger.dev) $\rightarrow$ Transforms data $\rightarrow$ Calls QuickBooks API.
* **Common patterns:** Event-driven (Webhooks) is preferred over Polling for real-time needs. Polling is only used if the source system lacks webhooks or has strict rate limits that make webhooks unreliable.
* **Trade-offs:** Webhooks are faster but require your server to be highly available and handle duplicate deliveries (at-least-once delivery). Polling is slower and wastes API quota but is easier to make idempotent.
* **Security/Reliability:** Webhook signature verification is non-negotiable. Background queues prevent API timeouts from dropping events.
* **Relation to your stack:** You will use Next.js API routes as the webhook receiver, Prisma to log the event, and Inngest to handle the asynchronous QuickBooks API call, ensuring the Next.js serverless function returns a `200 OK` immediately to HubSpot.
* **Interview questions:** *"How do you handle a scenario where HubSpot sends a webhook, but your server is temporarily down?"*  
  *Answer:* HubSpot retries webhooks with exponential backoff. If it permanently fails, we rely on a nightly polling fallback or a manual "Resync" button in the admin UI.
* **Interview-ready explanation:**  
  > "I start with a sequence diagram and an edge-case table before any code. For HubSpot to QuickBooks, the webhook handler only verifies the signature, stores the raw event and hands it to Inngest, then returns 200 well inside HubSpot's 5-second timeout. The background function fetches the latest contact instead of trusting the payload, so out-of-order events don't matter, and an idempotency key plus one-job-per-contact concurrency means HubSpot's retries never create a duplicate customer. Merges are flagged for Finance rather than merged automatically, and a nightly reconciliation catches anything missed if we're down longer than HubSpot's 24-hour retry window. My first draft made HubSpot wait for QuickBooks; reviewing it is how I learned to always acknowledge first and work later."

---

## Business Knowledge
* **Business problem:** Sales reps close deals in HubSpot, but finance has to manually re-type that data into QuickBooks to generate invoices. This causes delays, typos, and frustrated customers.
* **Who is affected:** Sales ops (delayed onboarding), Finance (manual data entry), Customers (delayed invoices).
* **Cost of doing nothing:** 2–3 hours/week of manual data entry per admin, plus the hidden cost of billing errors and delayed cash flow.
* **Expected benefits:** Instant invoicing, zero manual data entry, 100% data consistency between sales and finance.
* **Simple explanation:**  
  > "When a salesperson marks a contact as a 'Customer' in HubSpot, the system will automatically and instantly create that exact same customer in QuickBooks, ready for billing."
* **Questions to ask the business owner:**
  * *"What specific HubSpot property change triggers a customer to be 'ready' for QuickBooks?"*
  * *"Do we need to sync historical customers, or only new ones going forward?"*
* **Risks/Limitations:** This is a unidirectional sync (HubSpot $\rightarrow$ QB). If someone changes an email in QuickBooks, it will not update in HubSpot. (This is a deliberate, safe limitation for Phase 1).
* **Building trust:** Show them the sequence diagram. Say: *"I map this out first so we both agree on exactly what 'done' looks like before I write a single line of code."*
* **Client-facing explanation:**  
  > "When a salesperson marks a contact as a customer in HubSpot, that customer appears in QuickBooks within a minute, ready for billing, with no retyping. The same customer is never created twice, merged contacts go to your finance team to review, and if QuickBooks is down the system retries on its own. Anything it can't fix shows up on a dashboard with a one-click retry, and every night it double-checks that nothing was missed."
* **Objections & Responses:**
  * **Objection:** *"Can't we just use Zapier?"*
  * **Response:** *"Zapier is great for simple, low-volume tasks. But for core financial data, we need custom error handling, audit logs, and data transformation that Zapier cannot provide without becoming fragile and expensive at scale."*

---

## Interview and Client Notes
* **Technical Interview:** Emphasize decoupling. *"I never call external APIs directly from a webhook handler. I enqueue the job and return 200 OK immediately."*
* **Why the design holds up in an interview** (see [us-01-integration-design.md](us-01-integration-design.md)):
  * **At-least-once delivery:** HubSpot can send the same event more than once. An idempotency key (hash of the event fields, not the retry counter) makes Inngest and the database ignore duplicates, so no duplicate customers.
  * **Don't trust the payload:** the function fetches the latest contact from HubSpot, because webhooks carry ids and changed fields only. This also makes out-of-order events harmless.
  * **Respect financial data:** merged contacts are never merged in QuickBooks automatically; an orphaned record is flagged for Finance. B2B data cleanup has real financial consequences and shouldn't be automated blindly.
* **Discovery Call:** *"Walk me through the exact moment a contact becomes a billable customer today. What fields are absolutely required?"*
* **Proposal:** Include the sequence diagram as an appendix. It visually proves you understand their system.

### Client objections: practised answers
From the `/drill biz us-01` session on 2026-09-24 (practice persona: Dana, CFO of a 180-person distributor). Answer each part of the question in the order it was asked, and end with a question that moves toward a decision.

**1. "Why isn't this just a $30/month Zapier zap?"**
> "For simply copying new customers, Zapier is fine, and I'd tell you so. The trouble starts with the special situations. Say a salesperson marks a contact as a customer, changes it back, then marks it again: a simple zap creates two customer records. That company's invoices end up split across both, so your unpaid-invoices report is wrong at month-end, and someone spends time finding and merging them. Merged contacts in HubSpot cause the same mess, and when a zap fails, the error lands in one person's inbox. My version never creates the same customer twice, sends merges to Finance for review, and shows every failed sync on a dashboard with a retry button. Zapier can be set up to handle some of this, but every rule is built and maintained by hand, and nobody owns it when it breaks. How many duplicate customers do you have in QuickBooks today?"

**2. "What does it cost, and when do I get my money back?"**
> "Honestly, if we only count Priya's time, 2–3 hours a week at about €40 an hour, that's roughly €5k a year, so this doesn't pay back on time savings alone. The bigger value is preventing costly mistakes and getting paid sooner. Can I ask you three things? How many duplicate customers did you clean up last quarter? How many days pass between a deal closing and the first invoice? Has an invoice ever been missed because the customer wasn't in QuickBooks? One missed €8k invoice outweighs a year of Priya's time. So I'd start small: new customers only, a fixed price of €[X], live in about three weeks. That sync is also the foundation: the next automations, like invoice processing, reuse it and cost less."

**3. "Who fixes it after you've moved on, how fast, and what does it cost?"**
> "Three things. First, you own everything: the code and all accounts are in your company's name, with documentation any developer can pick up, so you're never dependent on me. Second, most problems don't need a developer: failed syncs appear on a dashboard, and Marco can press Replay once the cause is fixed, for example when QuickBooks maintenance ends. Third, for real problems I offer a monthly maintenance plan at €[Y] a month: a response within 4 business hours, the same day during month-end close, and I keep up with HubSpot and QuickBooks changes. Would that cover your month-end risk?"

**4. "Where does our data live, who can see it, and what if you're hacked?"**
> "Your data stays in your company's own accounts, in a region you choose, such as the EU. We store only what's needed: which HubSpot contact matches which QuickBooks customer, and a sync history. Only named people can see it, each with their own login, and old records are deleted after a period we agree. HubSpot access is read-only. QuickBooks access covers your accounting file, but our system only works with customer records, and you can switch the connection off from inside QuickBooks. Suspicious requests are rejected and every sync is recorded. No system is unbreakable, so we limit what an attacker could reach. Does that cover what your board will ask?"

**5. "Is AI making decisions about our financial records?"**
> "No. This project uses fixed rules that your finance team approves line by line. The same input always produces the same result. If a tax ID is invalid or missing, the sync stops and flags it: we never guess. AI may make sense later for reading PDF invoices, and even then a person approves the result before anything reaches QuickBooks."

**Discovery questions to ask back**
- How many duplicate customers were cleaned up last quarter, and how long did each take?
- How many days pass between a deal closing and the customer's first invoice?
- Has an invoice ever been missed or sent late because the customer wasn't in QuickBooks?
- How long does month-end close take, and how much of that is fixing customer data?
- When a sync goes wrong today, who notices, and how long does it take?

**Numbers sheet (prepare before every call)**

| Number | Value |
|---|---|
| Manual entry cost | 2–3 h/week × €40/h ≈ €5k/year (estimate) |
| Cleanup per duplicate customer | ~30 min (estimate) |
| First step: new-customer sync | €[X] fixed price, live in ~3 weeks (estimate), to decide |
| Maintenance plan | €[Y]/month, to decide |
| Response time | 4 business hours; same day during month-end close (proposal) |
