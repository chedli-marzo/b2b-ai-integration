# B2B AI Integration Platform

Portfolio project and learning vehicle. Owner is a senior full-stack JS engineer becoming a B2B AI integration consultant. Every feature must be:
1. production-grade,
2. explainable in a technical interview,
3. sellable to a non-technical business owner.

## Map
- `docs/00-index.md`: 5-phase blueprint (goal, gotcha, pro-tip, stack per phase).
- `docs/phase-n-*/sprint-xx-*/us-yy-<name>.md`: user stories, the unit of work. Format reference: `docs/phase-1-crm-erp-integration/sprint-01-business&architecture/us-01-hubspot-quickbooks-flow.md`.
- Naming: all files and folders lowercase kebab-case (except `CLAUDE.md`, `SKILL.md`). User stories: `us-NN-<max-3-words>.md`, words reflect the story, e.g. `us-02-data-mapping-sot.md`.

## Stack
- App: Next.js (App Router) only: UI plus API route handlers, one app. Express and FastAPI are deferred to Phase 6.
- Real-time (Phase 5 approvals): SSE or a managed realtime service. Serverless route handlers can't host a WebSocket server.
- DB: PostgreSQL + Prisma; pgvector from Phase 3.
- Jobs: Inngest or Trigger.dev, undecided. Decide in Phase 1 Sprint 02 and record it in that story's As Built section.
- Redis (cache, rate limiting), Zod, Vercel AI SDK, MCP TypeScript SDK.
- Fast-moving libs (AI SDK, Inngest, Trigger.dev, Prisma, MCP SDK): check current docs before writing code. Don't trust memory.

## Rules
- Pro-Tips in `00-index.md` and Acceptance Criteria in US files are requirements, not YAGNI: idempotency, retries, signature verification, audit logs, HITL. Everything beyond them stays minimal.
- Advanced ideas outside the current story go into `docs/phase-6-advanced-next-steps/discovery-backlog.md` under "Discovered while building", not into the code.
- Non-trivial change: plan first, explain the why and the trade-offs before writing code.
- If a library does the key trick (e.g. Inngest idempotency), explain what it does underneath. The owner must be able to explain it without the library.
- Everything under `docs/` is written in normal full English, never terse or caveman style. Business sections contain zero jargon.
- Tick an acceptance-criteria checkbox only after it is verified (test run or command output shown).

## Workflow
`/story <sprint>` writes the next user stories → `/build <US>` implements one and appends As Built notes → `/drill tech|biz` practices interviews and client calls.

Current: Phase 1, Sprint 01 (design docs, no code yet).
