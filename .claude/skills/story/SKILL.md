---
name: story
description: Write new user story files in this repo's US format for a sprint. Use when the user says /story or asks to write or plan user stories for an empty sprint.
argument-hint: <sprint folder or topic>
disable-model-invocation: true
---

Write user stories for: $ARGUMENTS

1. Read `docs/00-index.md` (phase goal, gotcha, pro-tip), then the format reference in `docs/phase-1-crm-erp-integration/sprint-01-business&architecture/`: `us-01-hubspot-quickbooks-flow/` (`00-index.md` and `us-01-knowledge.md`). Each story is a folder holding two files with exactly those section structures: `00-index.md`, the story (User Story, Acceptance Criteria, Deliverables, Git, Knowledge link, Practice Task; Git = story branch plus a numbered list of suggested commits, one per deliverable or logical step, naming per CLAUDE.md) and `us-NN-knowledge.md` (Technical Knowledge, Business Knowledge, Interview and Client Notes).
2. Read the existing US files in that phase. Continue numbering from the highest US number in the phase; don't overlap their scope. Folder name: `us-NN-<max-3-words>`, lowercase kebab-case, words reflect the story.
3. Scope: one story is 1–3 days of work with testable acceptance criteria. Propose the list of story titles first and wait for approval before writing files.
4. Stack follows CLAUDE.md (Next.js only, API as route handlers).
5. Business Knowledge (in the knowledge file): any number you did not measure is labelled "(estimate)". Client-facing lines contain no jargon.
6. Create or update the sprint's `00-index.md`, following `docs/phase-1-crm-erp-integration/sprint-01-business&architecture/00-index.md`: sprint branch, sprint goal, stories table, then acceptance criteria for the business owner (no jargon) and for a junior dev.
7. Write in normal full English.
