## User Story 1: Map HubSpot → QuickBooks Customer Workflow

### User Story
**As an** Integration Consultant,  
**I want to** map the end-to-end customer creation workflow from HubSpot to QuickBooks,  
**So that** we establish a clear, agreed-upon blueprint before writing code, preventing scope creep and architectural rework.

### Acceptance Criteria
* [ ] A visual sequence diagram exists showing the exact flow from HubSpot trigger to QuickBooks confirmation.
* [ ] The decision between Webhook-driven (event-based) vs. Polling (schedule-based) is explicitly documented with justification.
* [ ] Edge cases (e.g., customer creation, customer update, customer merge in HubSpot) are explicitly mapped.
* [ ] The business owner has reviewed and signed off on the workflow diagram.

### Deliverables
1. Mermaid.js or Excalidraw sequence diagram of the integration flow.
2. A 1-page "Integration Design Document" (IDD) outlining the chosen architecture.

---

### Technical Knowledge

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
  > "I always start with a sequence diagram. For HubSpot to QuickBooks, I use an event-driven webhook architecture decoupled by a durable execution engine like Inngest. This ensures the Next.js endpoint acknowledges receipt instantly, while the heavy lifting of data transformation and external API calls happens asynchronously, guaranteeing delivery even if QuickBooks experiences transient downtime."

---

### Business Knowledge

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
  > "We will build a secure, automated bridge. When a contact reaches a specific stage in HubSpot, our system securely transfers their details to QuickBooks in the background, eliminating manual entry."
* **Objections & Responses:**
  * **Objection:** *"Can't we just use Zapier?"*
  * **Response:** *"Zapier is great for simple, low-volume tasks. But for core financial data, we need custom error handling, audit logs, and data transformation that Zapier cannot provide without becoming fragile and expensive at scale."*

---

### Practice Task
Create a Mermaid.js sequence diagram showing: HubSpot User $\rightarrow$ HubSpot Webhook $\rightarrow$ Next.js API Route (Signature Check) $\rightarrow$ Inngest Queue $\rightarrow$ Data Transformation $\rightarrow$ QuickBooks API $\rightarrow$ Prisma Audit Log. Export this as a PNG and put it in your portfolio README.

---

### Interview and Client Notes
* **Technical Interview:** Emphasize decoupling. *"I never call external APIs directly from a webhook handler. I enqueue the job and return 200 OK immediately."*
* **Discovery Call:** *"Walk me through the exact moment a contact becomes a billable customer today. What fields are absolutely required?"*
* **Proposal:** Include the sequence diagram as an appendix. It visually proves you understand their system.

---