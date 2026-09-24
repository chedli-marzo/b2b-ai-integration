---
name: build
description: Implement one user story against its acceptance criteria, then append As Built notes to the story's 00-index.md. Use when the user says /build or asks to implement a US file.
argument-hint: <US path or id, e.g. "p1 us-04">
disable-model-invocation: true
---

Implement: $ARGUMENTS

1. Read the story's `00-index.md` and CLAUDE.md. List the acceptance criteria. If anything is ambiguous or conflicts with the CLAUDE.md stack, ask before coding.
2. Plan: files to touch, and each key decision with one rejected alternative and why. Wait for approval.
3. Work on the story's branch from its Git section (create it from the sprint branch if missing). Build in small steps. After each step, one line: which concept it demonstrates, and the proposed commit message (from the story's list, adjusted to what was actually done). Commit when the user approves.
4. If the story centres on a core concept (idempotency, retries/backoff, signature verification, schema-validation retry loop, hybrid search, agent loop, HITL approval), offer to let the user write that core function first, then review their version.
5. Verify every acceptance criterion with a test, curl or script run and show the output. Tick `[x]` only for verified criteria.
6. Append an `### As Built` section to the story's `00-index.md` using the template below. Normal full English. Only facts from this work: no generic textbook filler.
7. Rewrite the "Interview-ready explanation" and "Client-facing explanation" in `us-NN-knowledge.md` from what was actually built.
8. Update the `Current:` line in CLAUDE.md.

### As Built template

```markdown
### As Built

* **Code:** links to the main files, e.g. [webhook.ts](../../../../src/webhook.ts)
* **Decisions:** what was chosen, what was rejected, why (one line each).
* **Deviation from plan:** where the sections above turned out wrong or changed, and why.
* **War story:** the real problem hit during this story: situation → how it was found → fix → result. This is the best interview material.
* **Numbers:** anything measured (latency, retries, duplicates prevented, test count). Proof for clients.
```
