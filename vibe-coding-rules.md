---
name: vibe-coding-rules
version: "1.1.0"
description: >
  23-area checklist for security and engineering quality in AI-generated apps.
  Covers secrets, rate limiting, input validation, auth, SQL injection, CORS,
  security headers, file uploads, error handling, dependencies, XSS, deploy
  gates, AI/LLM risks, architecture, scalability, cost management, testing,
  data integrity, UX/accessibility, maintainability, legal/compliance,
  operational readiness, and AI-specific reliability pitfalls.
triggers:
  - user asks to review code for security or quality
  - user asks "is this safe to ship" or "production ready"
  - user wants a pre-deploy checklist
  - user is building an app/API/SaaS with AI help
  - user mentions vibe coding, code audit, security review, or technical debt
---

# Vibe Coding Rules — Security & Engineering Checklist

A 23-area standard for anything an AI tool generates: code, APIs, backend services, or full apps. Use it to **generate code correctly the first time**, to **review/audit existing code**, or to **run a pre-deploy gate check**.

The checklist is split into three parts:
- **Part 1 — Security (areas 1–13):** secrets, rate limiting, input validation, auth, SQL security, CORS, HTTP headers, file uploads, error handling, dependencies, XSS, deployment gate, AI/LLM-specific risks.
- **Part 2 — Engineering & Product (areas 14–23):** architecture, scalability, cost management, testing, data integrity, UX/accessibility, maintainability, legal/compliance, operational readiness, AI-specific code reliability pitfalls.
- **Part 3 — Architecture, Modularity & Documentation (every app):** the universal architecture, reusability, and documentation standard that applies to *any* app, plus the correct architecture per app type.

Full rule-by-rule detail — code examples and copy-paste prompts for all 23 areas — lives in `references/full-checklist.md`. Read that file before doing a thorough review or generating a `CLAUDE.md`/`.cursorrules` file. If that file is not present in the project, apply the rules below using the descriptions in this document.

## How to use this skill

**When generating new code (a feature, an endpoint, a full app):**
Apply the relevant rules from the Quick Reference table below proactively as you write. Priority order: always apply areas 1, 9, 11 (secrets, error handling, XSS); apply areas 2–8 when there is a server endpoint; apply area 5 when there is a database; apply areas 13–23 for anything beyond a trivial script.

**When reviewing or auditing existing code:**
Work through all 23 areas systematically. Report findings grouped by area (Security first, then Engineering/Product), citing the specific rule violated and the fix. Report non-security issues too — the user wants architecture, scalability, cost, testing, data integrity, UX, maintainability, legal, and operational findings, not just security.

**When asked for a pre-deploy / "is this safe to ship" check:**
Walk through the Quick Reference table as a gate checklist. Security items (areas 1–13) are **blockers**. Engineering/Product items (areas 14–23) are **strong recommendations**.

**When asked to generate a `CLAUDE.md` or `.cursorrules` file:**
Pull the "Drop-in prompt" blocks from `references/full-checklist.md` for the areas relevant to the project's stack and assemble them into the file.

## Quick Reference — All 23 Areas

| # | Area | Core Rule | Tools / Examples |
|---|---|---|---|
| 1 | Secrets | Keys in `.env` only. Never in frontend code. | `.gitignore`, `.env.example` |
| 2 | Rate Limiting | 5 req/15 min auth. 60 req/min general API. | express-rate-limit, slowapi |
| 3 | Input Validation | Server-side only. Schema validation required. | Zod, Pydantic |
| 4 | Auth | bcrypt min cost 12. JWT short expiry. httpOnly cookies. | NextAuth, Clerk, lucia-auth |
| 5 | SQL Security | ORM or parameterized queries only. No string concat. | Prisma, Drizzle, SQLAlchemy |
| 6 | CORS | No wildcard in production. Explicit origin whitelist. | `cors()` with origin config |
| 7 | HTTP Headers | CSP, HSTS, X-Frame-Options: DENY. | helmet (Node), django-csp |
| 8 | File Uploads | MIME + extension validation. UUID rename. Outside web root. | multer, S3, Cloudinary |
| 9 | Error Handling | Generic messages to client. Full context in logs. | Sentry, Datadog, Logtail |
| 10 | Dependencies | Audit after every install. Pin versions in production. | npm audit, pip-audit |
| 11 | XSS | No `dangerouslySetInnerHTML`. No `eval()`. No inline scripts. | DOMPurify |
| 12 | Deploy Gate | Confirm: secrets out · prod env vars set · HTTPS+HSTS · security headers · CORS locked · rate limits on · server-side validation on · audit clean · generic errors · backups+rollback ready · monitoring on. Any unmet security item blocks the deploy. | Pre-ship checklist |
| 13 | AI / LLM Security | Sanitize input. Server-side keys. Token budgets. | Server proxy, `max_tokens` |
| 14 | Architecture | Layered structure. One pattern per concern. | Services, repositories |
| 15 | Scalability | Design for 100x data. No N+1, paginate, index, cache. | Redis, DB indexes |
| 16 | Cost Management | Cap paid API usage per user. Monitor spend. | Usage logs, billing alerts |
| 17 | Testing | Unit tests on critical paths. CI before deploy. | Jest, Pytest, CI pipeline |
| 18 | Data Integrity | Transactions for multi-step writes. DB constraints. | Prisma transactions |
| 19 | UX / Accessibility | Loading/empty/error states. Semantic HTML. | Design system, ARIA |
| 20 | Maintainability | README, descriptive naming, pinned versions. | README.md, lockfiles |
| 21 | Legal / Privacy | Privacy policy. Data deletion flow. License checks. | GDPR/CCPA basics |
| 22 | Operational Readiness | Monitoring, backups, rollback plan before launch. | Sentry, automated backups |
| 23 | AI Code Reliability | Verify AI-used APIs exist. Check for deprecation. | Official docs review |

## Part 3 — Architecture, Modularity & Documentation

Security and the engineering checklist tell you *what not to ship*. This part tells you *how to shape the codebase* so it stays reviewable, reusable, and handoff-ready. Apply it to **every** app the AI generates.

### The universal standard (applies to any app)

**A. Architecture**
- **Layered separation.** Keep presentation, business logic, and data access in distinct layers. Transport (HTTP/UI/CLI) lives at the edge and never leaks into business logic.
- **Dependencies point inward.** UI/transport depends on logic; logic does not depend on UI/transport (ports & adapters / clean architecture). The core is framework-agnostic.
- **One pattern per concern.** Pick one approach for state, one for data fetching, one for errors — don't mix three.
- **Explicit boundaries.** Each module exposes a small public interface; internals stay private. Features compose modules, they don't reach into each other's guts.
- **Config via environment, not code.** Twelve-factor: no hardcoded URLs, keys, or client-specific values.

**B. Modularity & reusability**
- **Single responsibility.** A function/component/module should do one thing and be nameable in one phrase. If you need "and" to describe it, split it.
- **Extract on the second occurrence.** When logic appears twice, pull it into a shared unit (util, hook, service, middleware, package). Don't copy-paste.
- **Composable primitives.** Build a small set of building blocks (UI components, hooks, repositories, middlewares) and assemble features from them rather than writing each feature top-to-bottom.
- **Depend on interfaces, not implementations.** Abstract the swappable thing (DB, LLM provider, payment gateway) behind an interface so it can change without rippling through the codebase.
- **No god-files.** Split anything mixing more than ~2 concerns or growing past a screenful of unrelated logic. Co-locate related files; use barrel/`index` exports for clean import paths.
- **Separate data from presentation, and logic from view.** Content/config in data modules; reusable logic in hooks/services; views stay thin.

**C. Documentation**
- **README is mandatory:** what it is, how to run, scripts, required env vars, and a one-paragraph architecture overview.
- **Necessary comments only.** Explain the non-obvious *why*, never the obvious *what*. Delete comments that restate code.
- **Doc headers on modules/exports:** one line stating responsibility, plus inputs/outputs for non-trivial functions.
- **`ARCHITECTURE.md` for non-trivial apps:** the request/render pipeline, a "where do I change X" folder map, and the system boundaries.
- **Keep docs in sync with code.** Update docs in the *same commit* as the change — stale docs are worse than none.

### Correct architecture by app type

| App type | Layering (edge → core) | Must-haves |
|---|---|---|
| **Frontend-only** | routing → pages → feature components → reusable UI primitives → hooks (logic) → services (API client) → utils | Data/content separated from views; logic in hooks; one design-token source; route-level code-splitting; error boundary; client validation is UX-only (if a backend exists, it re-validates server-side) |
| **Backend-only** | routes/controllers → services (business logic) → repositories (data access) → models | Thin controllers, fat services; validate every input at the boundary; ORM/parameterized only; cross-cutting concerns as middleware (auth, rate-limit, logging); pagination + transactions + indexes |
| **Full-stack** | frontend layers ⟂ backend layers, joined by a typed contract | Single source of truth for the API shape (shared types / OpenAPI / schema package); shared code in a real boundary, not copy-paste; each unit independently deployable; auth + CORS + server-side validation end-to-end |
| **AI / LLM app** | UI → app logic → **LLM service (provider behind an interface)** → provider SDK | Keys server-side only (browser hits *your* proxy); treat model output as untrusted (parse/validate/sanitize); token & cost caps (`max_tokens`, per-user budgets, caching, backoff); prompts versioned as assets + evals; pin model IDs; verify the SDK method actually exists |
| **MCP server** | transport (stdio/HTTP) → tool registry → **individual tools** → domain logic | Each tool = one small, well-named function with a strict input schema and clear output contract; validate inputs (untrusted); least privilege; transport separate from tool logic (unit-testable); document each tool's purpose/args/side-effects; confirm destructive actions |
| **Forward-deployed app** (on-prem / per-client / edge) | same artifact across clients; **differences only in config** | Configuration-first (zero hardcoded client specifics); reproducible builds + pinned deps; self-contained & offline-tolerant (vendor/self-host critical assets, degrade gracefully); handoff docs + ops runbook (install/upgrade/rollback); secrets via the client's store; backups + rollback plan before launch |

**Notes on the trickier types**

- **AI / LLM apps.** The single most common failure is calling the provider straight from the UI with a baked-in key. Always route through a server proxy, wrap the provider in one swappable `llm` service, and treat every model response as untrusted input — schema-validate it before you act on it or render it.
- **MCP servers.** Think of each tool as a public API endpoint: strict input schema, least privilege, idempotent/safe where possible, and a clear error contract that never leaks internals. Keep the tool functions free of protocol code so they can be tested directly.
- **Forward-deployed apps.** "Forward-deployed" means the app runs in *someone else's* environment, so portability and handoff beat cleverness. One configurable artifact, no phone-home without consent, and documentation an operator who has never met you can follow.

## Reference files

- `references/full-checklist.md` — the complete 23-area guide with rule bullets, code examples, and drop-in prompts. Load this whenever you need specifics beyond the summary table above.

---

Based on the security checklist by [@tahajaffriii](https://github.com/tahajaffriii), expanded with engineering, product, and operational analysis.
