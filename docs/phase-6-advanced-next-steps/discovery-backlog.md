# Phase 6: Discovery & Advanced Next Steps

This is the parking lot for everything deliberately left out of Phases 1–5. Nothing here gets built before Phase 5 ships. While building Phases 1–5, any advanced idea that comes up is logged under "Discovered while building" instead of going into the code.

Every item names the **client trigger**: the situation where a client actually needs it. That trigger tells you when to pick the item up, and it is also the sentence you use to sell it.

---

## Deferred decisions

Alternatives rejected for now to keep Phases 1–5 simple.

| Topic | Why deferred | Client trigger |
|---|---|---|
| Express API as a separate service | Next.js route handlers cover Phases 1–5 with one app to build and deploy. | Long-running processes, a WebSocket server, scaling the API separately from the UI, or a client team that already runs Node services. |
| Python / FastAPI + Pydantic | One language (TypeScript) end to end while learning the concepts. | The client's data or AI team works in Python, or document processing needs Python-only libraries. |
| Temporal (instead of Inngest / Trigger.dev) | Heavier to run and learn; Inngest or Trigger.dev covers retries and idempotency. | Workflows that run for days or weeks, a strict self-hosting requirement, very high volume. |
| Self-hosted WebSockets | Serverless route handlers can't host a WebSocket server; SSE or a managed realtime service is enough for Phase 5 approvals. | Many concurrent live dashboards, or a managed realtime service is too costly or not allowed. |
| Self-hosted reranker (BGE) instead of Cohere | A managed API is faster to ship. | The client forbids sending documents to third-party APIs. |

---

## Advanced topics

### Reliability and scale
| Topic | What it is | Client trigger |
|---|---|---|
| Message queues (SQS, Kafka) | Dedicated event backbone between systems. | Thousands of events per minute, or many systems consuming the same events. |
| Multi-tenancy with Postgres Row-Level Security | One platform serving several clients, with the database itself enforcing data separation. | Running the platform as a product for several clients instead of one custom build each. |

### AI quality
| Topic | What it is | Client trigger |
|---|---|---|
| Evals and regression tests | A golden dataset of real documents with expected outputs, re-run on every prompt or model change. | "How do we know accuracy won't drop next month?" |
| LLM tracing and observability | Trace each AI call: prompt, output, tokens, cost, latency (OpenTelemetry, Langfuse). | Debugging a wrong answer in production, or monthly AI cost reporting. |
| Model routing and cost control | Cheap model for easy tasks, strong model for hard ones; caching; an AI gateway. | The AI bill becomes a line item the CFO asks about. |
| Open-weight / self-hosted models | Run models such as Llama or Mistral on infrastructure the client controls (vLLM, Ollama). | Data can't leave the client's environment (legal, health, finance). |

### Security and compliance
| Topic | What it is | Client trigger |
|---|---|---|
| SSO (SAML / OIDC) and fine-grained RBAC | Staff log in with the company identity provider; permissions per role and per action. | IT requires Okta or Microsoft Entra login. |
| PII redaction | Strip or mask personal data before it reaches an LLM. | GDPR, customer contracts, sensitive HR or health data. |
| Prompt-injection defence for agents | Treat tool output and documents as untrusted input; limit what an agent can do after reading them. | Agents read emails or documents from outside the company. |
| SOC 2 / GDPR readiness | Audit trails, retention policies, data processing agreements with LLM providers, EU data residency. | The client's security questionnaire arrives before signing. |

### Deployment
| Topic | What it is | Client trigger |
|---|---|---|
| Docker + client-owned cloud | Package the platform and deploy it into the client's AWS, Azure or GCP account. | "It has to run in our cloud." Common in the 50–500 employee market. |
| Infrastructure as code (Terraform) | Environments reproducible from code. | Handover to the client's IT team, or several environments per client. |

### More connectors
| Topic | What it is | Client trigger |
|---|---|---|
| More CRM / ERP connectors | Xero (sprint-061), NetSuite, Salesforce, Shopify, Microsoft Dynamics. | The next client runs a different stack. |
| Remote MCP servers with OAuth | Expose client tools as MCP servers that any approved agent can use. | The client wants its own AI assistants to use the same tools. |

### Consulting business
| Topic | What it is | Client trigger |
|---|---|---|
| Reusable accelerator | Turn the platform into a starter repo per engagement. | The second client: stop rebuilding the foundations. |
| Fixed-price packages | AI Readiness Audit → Phase 1 + 2 implementation → monthly retainer. | Clients want a known price before saying yes. |
| Case studies | Before/after numbers taken from the As Built notes. | Every sales conversation. |

---

## Discovered while building

Append here during Phases 1–5. Format:

`- **<topic>** (from p1 us-04): what came up, why it was out of scope, client trigger.`
