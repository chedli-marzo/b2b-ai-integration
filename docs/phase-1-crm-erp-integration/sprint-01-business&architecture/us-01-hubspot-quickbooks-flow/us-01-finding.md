# us-01 Findings: Review of the First Flow Diagram

**Reviewed:** the first version of the HubSpot → QuickBooks flow diagram ([readme/sequence-diagram.png](../../../../readme/sequence-diagram.png)).
**Verdict:** the diagram gets the normal case roughly right, but as drawn, the system would create **duplicate customers** and could **lose customers without anyone noticing**. It needs rework before it can be signed off.

![First version of the flow diagram](../../../../readme/sequence-diagram.png)

---

## How to read the diagram

- **Boxes along the top** are the people and systems involved: the salesperson, HubSpot, our system, the background task runner (Inngest), QuickBooks and our database.
- **Solid arrows** mean "asks the other to do something". **Dashed arrows** are replies.
- **The dotted "alt" box** means "either this happens, or that happens", depending on the condition written in brackets.
- Time runs **from top to bottom**.

## What the first diagram says, in plain words

1. A salesperson does something in HubSpot.
2. HubSpot sends our system an instant notification about it.
3. Our system checks the notification is genuine (a security check).
4. If it is genuine, the work is put on a to-do list handled in the background.
5. The background task reformats the customer data and sends it to QuickBooks.
6. QuickBooks writes an entry into our records.
7. Only after all of that is finished does our system tell HubSpot "received".
8. If the notification is not genuine, it is rejected.

### What is already right
- The security check happens first, so fake notifications are rejected.
- The work runs in the background through a to-do list, which is the right foundation for reliability.
- There is a record (audit log) of what was done.

---

## What needs to be fixed, and why

| # | Finding | Risk if not fixed | Priority |
|---|---|---|---|
| 1 | HubSpot is kept waiting until everything is done | Duplicate customers in QuickBooks | High |
| 2 | No record of which HubSpot contact matches which QuickBooks customer | Duplicates every time a customer's details change | High |
| 3 | Only one scenario is shown; special cases are missing | Leads end up in accounting; merged contacts create a mess | High |
| 4 | No plan for when something fails | Customers silently missing from QuickBooks | High |
| 5 | The incoming notification is saved too late | Nothing to trace or retry when something fails | Medium |
| 6 | Two HubSpot behaviours are not taken into account | Outdated or missed customer details | Medium |
| 7 | Some arrows show systems doing things they don't do | The blueprint misleads the developers | Medium |
| 8 | HubSpot gets two replies | Confusing blueprint | Low |
| 9 | The diagram's source text is not stored with the project | Hard to update and review | Low |

### 1. HubSpot is kept waiting until everything is done
**What the diagram shows:** our system only tells HubSpot "received" after QuickBooks has been updated and the result logged.
**Why it's a problem:** HubSpot only waits a few seconds. If QuickBooks is slow, HubSpot assumes the notification was lost and sends it again, and the customer gets created twice. Think of a courier who has to wait at your door while you unpack, cook and eat: they give up, leave, and deliver the same parcel again tomorrow.
**Fix:** sign for the parcel immediately. Our system confirms receipt to HubSpot the moment the work is safely on the to-do list, then does the rest in the background.

### 2. No record of which HubSpot contact matches which QuickBooks customer
**What the diagram shows:** QuickBooks never answers back, so we never learn the customer number QuickBooks assigned.
**Why it's a problem:** the next time that customer's details change in HubSpot, the system can't find the matching QuickBooks customer to update, so it creates a new one. This is the most common cause of duplicate customers in integrations.
**Fix:** when QuickBooks creates a customer, it replies with its customer number. We save it next to the HubSpot contact, like an address book linking the two systems. Before doing anything, the system checks this address book first.

### 3. Only one scenario is shown; special cases are missing
**What the diagram shows:** one generic "trigger event".
**Why it's a problem:** real life has several different situations, each needing different handling:
- **A contact who is not a customer yet** (a lead): must be ignored. Otherwise every new lead lands in your accounting software.
- **A brand-new customer:** create them in QuickBooks.
- **An existing customer whose details changed:** update them in QuickBooks, never create a second one.
- **Two duplicate contacts merged in HubSpot:** QuickBooks may now hold two customers for one real company. A rule is needed (see decisions below).

This story's acceptance criteria require these cases to be mapped.
**Fix:** the diagram shows each case as a separate branch.

### 4. No plan for when something fails
**What the diagram shows:** only the path where everything works.
**Why it's a problem:** QuickBooks goes down for maintenance, limits how many requests it accepts, and sometimes rejects data. With no plan, a customer can silently fail to appear in QuickBooks, and finance only finds out when an invoice can't be sent.
**Fix:** the system retries automatically, waiting a little longer each time. If it still fails, the sync is marked as failed, the right person is alerted, and a "Replay" button lets them retry once the problem is fixed. The full "what if it breaks" plan is story us-03.

### 5. The incoming notification is saved too late
**What the diagram shows:** our records are only written at the very end.
**Why it's a problem:** if anything fails along the way, there is no trace that the notification ever arrived, so nothing can be investigated or retried.
**Fix:** keep a copy of every notification the moment it arrives. That copy is what makes the "Replay" button possible.

### 6. Two HubSpot behaviours are not taken into account
**Why it's a problem:**
- HubSpot's notification only says *which* contact changed, not the contact's details. The system has to look up the latest details itself. Doing this also guarantees we always copy the most recent version, even when notifications arrive in the wrong order.
- HubSpot can bundle several changes into one notification. Each change has to be handled separately, or some get lost.

**Fix:** the diagram shows a "look up the latest contact details" step and one background task per change.

### 7. Some arrows show systems doing things they don't do
**What the diagram shows:** QuickBooks writing into our records, and "data transformation" drawn as if it were a separate system.
**Why it's a problem:** this diagram is the blueprint you sign off and the developers build from. If it describes something that can't happen, the build won't match what was agreed.
**Fix:** our system does all the asking. QuickBooks only answers, and our system writes its own records. Reformatting the data is one step inside our background task, not a separate system.

### 8. HubSpot gets two replies
**What the diagram shows:** a rejection inside the "not genuine" branch, then another reply at the very end.
**Fix:** exactly one reply per notification: "received" if genuine, "rejected" if not.

### 9. The diagram's source text is not stored with the project
**What we have:** the diagram was written as text (Mermaid) in an online editor, but only the exported image was saved in the project.
**Why it's a problem:** an image can't be edited easily, and changes between versions can't be compared.
**Fix:** store the Mermaid text with the other project documents, where it draws itself automatically. The image becomes an export for presentations. Also remove the editor-specific `theme: mc` setting, which other viewers don't recognise.

---

## The corrected flow, in plain words

1. A salesperson marks a contact as a customer in HubSpot.
2. HubSpot sends our system an instant notification.
3. Our system checks it is genuine. If not, it is rejected and nothing else happens.
4. Our system keeps a copy of the notification, puts one task per change on the to-do list, and **immediately** confirms receipt to HubSpot.
5. In the background, the task looks up the latest contact details from HubSpot and checks the data is complete and valid.
6. It checks the address book linking HubSpot contacts to QuickBooks customers, then:
   - **not a customer yet:** does nothing, and records that it skipped it;
   - **new customer:** creates them in QuickBooks and saves the new QuickBooks number in the address book;
   - **existing customer:** updates them in QuickBooks;
   - **merged contacts:** follows the rule you choose below.
7. Every result is written to the audit log.
8. If QuickBooks is unavailable, the task retries automatically. If it keeps failing, it is marked as failed, someone is alerted, and it can be replayed with one click.

---

## Decisions to make

**Decisions 1 and 2 were made on 2026-09-23:** see [us-01-integration-design.md](us-01-integration-design.md#decisions).

| # | Decision | Options | Recommendation | Who decides |
|---|---|---|---|---|
| 1 | At what exact moment is a HubSpot contact "ready for QuickBooks"? | a) Lifecycle stage becomes "Customer" · b) A deal is marked "Closed Won" · c) Another rule you use today | Whichever moment finance currently starts billing | Business owner |
| 2 | When two duplicate contacts are merged in HubSpot, what happens to the extra customer in QuickBooks? | a) Mark it inactive in QuickBooks automatically · b) Flag it for finance to review and handle by hand · c) Ignore it | **b) Flag for review.** It touches financial records (the extra customer may already have invoices), so a person should decide. Automate later once the pattern is clear. | Business owner + finance |
| 3 | Copy existing customers, or only new ones from now on? | a) New customers only · b) Also a one-off copy of existing customers | **a) for launch**, then b) as a separate one-off job if needed. Copying history needs its own cleanup of old duplicates. | Business owner |
| 4 | When a customer's details change in HubSpot, should QuickBooks be updated? | a) Always update · b) Update some details only (e.g. never touch the billing email) | Decided field by field in story us-02 ("source of truth") | Business owner + finance |
| 5 | Add a nightly safety check comparing HubSpot and QuickBooks, to catch any notification that was missed? | a) Yes · b) No | **a) Yes**, built in Sprint 04 (Reliability). Instant notifications are very reliable, but not perfect. | Consultant (owner informed) |

The diagram has been redrawn to match decisions 1 and 2 (v3 in [us-01-integration-design.md](us-01-integration-design.md)) and is ready for sign-off.
