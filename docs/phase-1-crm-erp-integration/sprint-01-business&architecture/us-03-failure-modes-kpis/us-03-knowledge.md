# us-03 Knowledge: Define Integration Requirements, Failure Scenarios, and Success Metrics

Story: [00-index.md](00-index.md)

---

## Technical Knowledge
* **What it is:** Planning for when things go wrong, because in distributed systems, they will go wrong.
* **Why it matters:** An integration that works 95% of the time but silently fails 5% of the time is worse than no integration at all, because it creates false confidence and data drift.
* **How it works:** Generate a `correlation_id` (UUID) on webhook receipt. Pass it to every log. If the QuickBooks API returns a 5xx, the background job retries with exponential backoff. If it returns a 4xx (bad data), it fails permanently and routes to a DLQ table, triggering a Slack alert.
* **Common patterns:** Circuit Breaker pattern (stop calling QuickBooks if it's returning 503s to save quota), Idempotency keys (ensuring retrying the same payload doesn't create two customers).
* **Trade-offs:** Aggressive retries can overwhelm a struggling downstream system. DLQs require manual or automated remediation processes.
* **Security/Reliability:** Logs must never contain PII (Personally Identifiable Information) or raw OAuth tokens. Use structured logging (e.g., Pino) for easy querying in Datadog or LogRocket.
* **Relation to your stack:** Next.js middleware or API routes can inject correlation IDs. Prisma will store the `sync_events` (success/fail). Inngest/Trigger.dev natively handles the retry/backoff logic.
* **Interview questions:** *"How do you debug a failed sync from three weeks ago?"*  
  *Answer:* I query the `sync_events` table by the customer's HubSpot ID or email, find the `correlation_id`, and trace that ID through our structured logs to see the exact API request and response payload.
* **Interview-ready explanation:**  
  > "I design for failure first. Every event gets a correlation ID. Transient failures trigger exponential backoff via Inngest. Permanent failures, like validation errors, route to a Dead Letter Queue in Postgres and trigger a Slack alert. This ensures zero silent failures and makes debugging trivial."

---

## Business Knowledge
* **Business problem:** When integrations break silently, finance doesn't know until a customer complains about not receiving an invoice.
* **Who is affected:** IT/Ops (firefighting), Finance (missing revenue), Management (lack of visibility).
* **Cost of doing nothing:** "Silent failures" lead to lost revenue, compliance issues, and a complete loss of trust in the automation.
* **Expected benefits:** Peace of mind. If something breaks, the right person is notified immediately with context, often before the business even notices.
* **Simple explanation:**  
  > "We are building a 'check engine' light for your data. If a sync fails, the system doesn't just give up; it tries a few times, and if it still can't, it immediately alerts us with exactly what went wrong so we can fix it."
* **Questions to ask the business owner:**
  * *"If a sync fails, who should be notified?"*
  * *"What is the maximum acceptable delay between a HubSpot update and a QuickBooks update?"*
* **Risks/Limitations:** No system is 100% uptime. We are designing for rapid recovery, not mythical perfection.
* **Building trust:** Show them the FMEA. *"I've already thought about what happens if QuickBooks goes down for maintenance, and here is exactly how the system will handle it safely."*
* **Client-facing explanation:**  
  > "We build enterprise-grade reliability into the foundation. This includes automatic retry mechanisms, detailed audit logs, and instant alerts if any data fails to sync, ensuring your financial operations never miss a beat."
* **Objections & Responses:**
  * **Objection:** *"This sounds like it will take a long time to build all this error handling."*
  * **Response:** *"Using modern durable execution tools like Inngest, 80% of this reliability is built-in. The time invested here prevents weeks of future debugging and data cleanup."*

---

## Interview and Client Notes
* **Technical Interview:** Use the phrase "Durable Execution" and "Dead Letter Queue." Mention that you never trust network calls to succeed on the first try.
* **Discovery Call:** *"What is your current process for finding out if a HubSpot-to-QuickBooks sync failed? How long does it usually take to find out?"*
* **Proposal:** Frame reliability not as a "technical feature," but as "Risk Mitigation and Operational Continuity."
* **Case Study:** *"Reduced silent data failures by 100% by implementing a Dead Letter Queue and correlation ID tracking, cutting debugging time from hours to seconds."*
