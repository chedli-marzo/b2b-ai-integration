## User Story 2: Define Customer Data Mapping and Source of Truth

### User Story
**As a** Lead Developer,  
**I want to** define the exact field-level data mapping and establish a strict "Source of Truth" (SoT),  
**So that** we prevent data corruption, duplication, and sync conflicts between systems.

### Acceptance Criteria
* [ ] A field-level mapping document exists (e.g., HubSpot `firstname` + `lastname` $\rightarrow$ QuickBooks `DisplayName`).
* [ ] The "Source of Truth" for each entity (Customer, Invoice) is explicitly declared (e.g., "HubSpot is SoT for contact info, QuickBooks is SoT for billing address").
* [ ] A Prisma schema is drafted to store the `integration_mappings` (linking HubSpot ID to QuickBooks ID).
* [ ] Data validation rules (e.g., Zod schema) are defined for the transformed payload before it hits QuickBooks.

### Deliverables
1. Data Mapping Spreadsheet (Source Field, Target Field, Transformation Logic, Required/Optional).
2. Prisma schema draft for `integration_mappings` and `sync_events`.
3. TypeScript Zod schema for the transformed QuickBooks payload.

### Git
* **Branch:** `story/p1-us-02-data-mapping-sot`, from `sprint/p1-s01-business-architecture`
* **Suggested commits:**
  1. `docs(p1-us-02): add field mapping table`
  2. `docs(p1-us-02): declare source of truth per field`
  3. `docs(p1-us-02): draft prisma schema for mappings and events`
  4. `docs(p1-us-02): draft zod schema for quickbooks customer`
  5. `docs(p1-us-02): record finance and sales ops sign-off`
  6. `docs(p1-us-02): add as-built notes`

---

### Knowledge
Technical, business, interview and client notes: [us-02-knowledge.md](us-02-knowledge.md)

---

### Practice Task
Draft a Prisma schema for an `integration_mapping` table and a TypeScript Zod schema that validates a HubSpot webhook payload and transforms it into a valid QuickBooks Customer creation payload.
