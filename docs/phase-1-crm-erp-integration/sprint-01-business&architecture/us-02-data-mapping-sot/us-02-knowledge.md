# us-02 Knowledge: Define Customer Data Mapping and Source of Truth

Story: [00-index.md](00-index.md)

---

## Technical Knowledge
* **What it is:** The explicit rules governing how data translates from one schema to another, and which system "wins" in a conflict.
* **Why it matters:** APIs rarely have 1:1 field matches. HubSpot might have `phone`, QuickBooks might require `PrimaryPhone.PrimaryTextPin`. Without strict mapping, data is lost or malformed.
* **How it works:** You extract the HubSpot payload, run it through a transformation function (validated by Zod), and format it to match the QuickBooks API specification. You then store the resulting `quickbooks_id` alongside the `hubspot_id` in your database.
* **Common patterns:** Unidirectional sync (easiest, safest). Bidirectional sync (requires conflict resolution strategies like "Last Write Wins" or "System of Record prioritization").
* **Trade-offs:** Strict mapping requires more upfront dev time but prevents catastrophic data corruption. Loose mapping (e.g., dumping everything into a generic "notes" field) is fast but useless for automation.
* **Security/Reliability:** Never trust incoming webhook data. Always validate with Zod/Pydantic before processing. Store external IDs securely to prevent orphaned records.
* **Relation to your stack:** Use Prisma to create an `integration_mappings` table. Use TypeScript and Zod to enforce strict typing during the transformation phase, catching errors before they reach QuickBooks.
* **Interview questions:** *"How do you handle a scenario where a customer is merged in HubSpot?"*  
  *Answer:* HubSpot sends a `contact.merge` webhook. The integration must detect this, find the old `integration_mapping`, and either update the QuickBooks record or flag it for manual review, as QuickBooks merge logic is complex.
* **Interview-ready explanation:**  
  > "I enforce a strict Source of Truth model. For customer creation, HubSpot is the master. I use Zod to validate and transform the payload into QuickBooks' exact schema before transmission. Crucially, I persist the mapping of hubspot_id to quickbooks_id in Postgres, which is the foundation for all future updates and idempotency."

---

## Business Knowledge
* **Business problem:** "Garbage in, garbage out." If sales enters "Acme Inc" in HubSpot, but finance needs "Acme Incorporated" for tax purposes, manual translation is currently happening.
* **Who is affected:** Finance (rejected invoices due to bad data), Sales (frustrated by rigid forms).
* **Cost of doing nothing:** Invoices get rejected by accounting software, delaying payment. Duplicate customer records are created in QuickBooks, messing up financial reporting.
* **Expected benefits:** Clean, standardized data in QuickBooks. No more duplicate customers.
* **Simple explanation:**  
  > "We will create a strict translation dictionary. The system will automatically format names, addresses, and tax IDs exactly as QuickBooks requires them, every single time."
* **Questions to ask the business owner:**
  * *"If a customer's email changes in HubSpot, should we overwrite it in QuickBooks, or is the QuickBooks email locked for billing?"*
  * *"What is the exact naming convention you use in QuickBooks?"*
* **Risks/Limitations:** We will only map the fields you explicitly approve. If a new custom field is added to HubSpot later, the integration will ignore it until we update the mapping.
* **Building trust:** Present the mapping spreadsheet. Say: *"I want to review this line-by-line with your finance team to ensure no critical data is lost in translation."*
* **Client-facing explanation:**  
  > "We build a secure translation layer. It takes your HubSpot data and formats it perfectly for QuickBooks, ensuring your financial records are always accurate and compliant."
* **Objections & Responses:**
  * **Objection:** *"Why can't it just figure it out automatically with AI?"*
  * **Response:** *"AI is great for unstructured data like PDFs, but for structured financial records, deterministic, rule-based mapping is 100% reliable, faster, and cheaper. We don't want AI guessing a tax ID."*

---

## Interview and Client Notes
* **Technical Interview:** Highlight the `integration_mapping` table. *"Without persisting the foreign key relationship between the two systems, updates and deletes are impossible to track reliably."*
* **Discovery Call:** *"Let's look at your top 5 most recent QuickBooks customers. What fields were missing or formatted incorrectly when they came from HubSpot?"*
* **Proposal:** Specify that *"Data Mapping is a collaborative phase requiring sign-off from both Sales Ops and Finance."*
