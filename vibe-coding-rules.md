---
name: vibe-coding-rules
version: "2.0.0"
author: "Muhammad Hassaan ul Mustafa (FAST-NUCES)"
description: >
  31-area production-readiness standard for AI-generated apps, APIs, backends,
  agents, and MCP servers. Part 1 — Security (1–18): secrets+rotation, rate
  limiting, input validation, auth+RBAC/IDOR, SQL, CORS, headers, uploads, error
  handling, dependencies+supply-chain, XSS, deploy gate, AI/LLM risks (direct +
  indirect prompt injection, tool-use authz, PII redaction, cost attacks), SSRF,
  CSRF, webhook/inbound security, logging+PII, secret scanning. Part 2 —
  Engineering & Product (19–31): architecture, scalability+caching, cost,
  testing, data integrity+idempotency, UX/accessibility (WCAG 2.2 AA),
  maintainability, legal/privacy, ops readiness+observability, AI reliability,
  CI/CD gates, threat modeling, resilience, safe rollout. Part 3 — Architecture,
  Modularity & Documentation per app type.
triggers:
  - user asks to review code for security or quality
  - user asks "is this safe to ship" or "production ready"
  - user wants a pre-deploy checklist
  - user is building an app/API/SaaS with AI help
  - user is building an agent, MCP server, scraper, or anything that fetches URLs
  - user mentions vibe coding, code audit, security review, or technical debt
---

<!--
  vibe-coding-rules v2.0.0 — 2026-07-01
  Original security checklist (areas 1–13) by Taha Jaffri (@tahajaffriii).
  v2.0 expansion: added security areas 14–18 (SSRF, CSRF, webhook/inbound,
  logging+PII, secret scanning), engineering areas 28/30/31 (resilience, CI/CD
  gates, threat modeling+safe rollout), depth on 1/4/10/13, and the previously
  missing references/full-checklist.md. Part 3 (architecture/modularity) kept
  from v1.1. See CHANGELOG.md for the full diff and rationale.
-->

# Vibe Coding Rules — Security & Engineering Checklist (v2.0)

A **31-area production-readiness standard** for anything an AI tool generates: code, APIs, backend services, agents, MCP servers, or full apps. Use it to **generate code correctly the first time**, to **review/audit existing code**, or to **run a pre-deploy gate check**.

Three parts:
- **Part 1 — Security (areas 1–18):** secrets, rate limiting, input validation, auth, SQL, CORS, HTTP headers, file uploads, error handling, dependencies/supply-chain, XSS, deploy gate, AI/LLM risks, **SSRF, CSRF, webhook/inbound security, logging & PII, secret scanning**.
- **Part 2 — Engineering & Product (areas 19–31):** architecture, scalability, cost, testing, data integrity, UX/accessibility, maintainability, legal/privacy, operational readiness, AI reliability, **CI/CD gates, threat modeling, resilience, safe rollout**.
- **Part 3 — Architecture, Modularity & Documentation (every app):** the universal architecture/reusability/documentation standard, plus the correct architecture per app type.

Full rule-by-rule detail — code examples and copy-paste prompts for all areas — lives in `references/full-checklist.md`. Read it before a thorough review or before generating a `CLAUDE.md`/`.cursorrules` file. If that file is absent, apply the rules below from the descriptions here.

## How to use this skill

**When generating new code:** apply the relevant Quick-Reference rows proactively. Always apply areas 1, 9, 11 (secrets, error handling, XSS); apply 2–8 when there's a server endpoint; apply 5 with a database; apply **14 (SSRF) whenever the app fetches a URL** and **15 (CSRF) on cookie-auth state changes**; apply 19–31 for anything beyond a trivial script.

**When reviewing/auditing:** work through all 31 areas systematically. Report findings grouped by area (Security first), citing the specific rule + fix. Include non-security findings too.

**Pre-deploy / "is this safe to ship":** walk the Quick Reference as a gate. Security items (1–18) are **blockers**. Engineering/Product items (19–31) are **strong recommendations**.

**Generating `CLAUDE.md`/`.cursorrules`:** pull the drop-in prompt blocks from `references/full-checklist.md` for the project's stack and assemble them.

**Project-type focus:** for a **scraper/agent/MCP server**, prioritize 13 (treat fetched/scraped content as untrusted — indirect prompt injection), 14 (SSRF), 17 (logging/PII), 28 (resilience). Frontend-only: 1, 11, 17. Public API: 2, 3, 4, 6, 16, 30.

## Quick Reference — Part 1 · Security (1–18)

| # | Area | Core Rule | Tools / Examples |
|---|---|---|---|
| 1 | Secrets & rotation | `.env`/secret manager only. Never in frontend, logs, or errors. Rotate + scope keys. | gitleaks, Vault, Doppler |
| 2 | Rate Limiting | 5/15min auth, 60/min API, 10/min LLM, per-user + per-IP. 429 + Retry-After. | express-rate-limit, slowapi |
| 3 | Input Validation | Server-side schema validation. Allowlist, not denylist. | Zod, Pydantic |
| 4 | Auth & access | bcrypt≥12/argon2, short JWT, httpOnly refresh. Check **ownership (IDOR)** + role every request. MFA + lockout + safe reset. | NextAuth, Clerk, lucia-auth |
| 5 | SQL Security | ORM/parameterized only. Least privilege. No leaked DB errors. | Prisma, Drizzle, SQLAlchemy |
| 6 | CORS | No wildcard in prod. Explicit origin allowlist. | `cors()` with origin |
| 7 | HTTP Headers | CSP, HSTS, X-Frame-Options: DENY, nosniff, Referrer-Policy. | helmet, django-csp |
| 8 | File Uploads | MIME+ext+size server-side. UUID rename. Off web root. | multer, S3 |
| 9 | Error Handling | Generic to client, full context to logs. Correct status codes. | Sentry, Logtail |
| 10 | Dependencies & supply chain | Audit + pin + lockfile integrity. Block untrusted postinstall. Dependabot/SBOM. | npm audit, pip-audit |
| 11 | XSS | No `dangerouslySetInnerHTML`/`eval`/`innerHTML` with user data. | DOMPurify |
| 12 | Deploy Gate | Run the pre-ship gate; any unmet security item blocks deploy. | Pre-ship checklist |
| 13 | AI / LLM Security | Server-side keys, `max_tokens`, per-user budget. **Untrusted input AND fetched/scraped content (direct + indirect prompt injection).** Tool-use authz. Redact PII. Sanitize output. | server proxy |
| 14 | **SSRF** | Allowlist outbound hosts/schemes. Block private/link-local IPs + cloud metadata (169.254.169.254). Re-check resolved IP. | ip guard |
| 15 | **CSRF** | SameSite cookies + anti-CSRF token on state-changing requests. | csurf, SameSite |
| 16 | **Webhook/inbound** | Verify HMAC signature, constant-time compare, replay protection. | — |
| 17 | **Logging & PII** | Structured logs, secrets/PII redacted, retention + access control. | OTel, structlog |
| 18 | **Secret scanning** | Pre-commit + CI scan; fail on hit; rotate anything leaked to history. | gitleaks, trufflehog |

## Quick Reference — Part 2 · Engineering & Product (19–31)

| # | Area | Core Rule | Tools / Examples |
|---|---|---|---|
| 19 | Architecture | Layered structure. One pattern per concern. | Services, repositories |
| 20 | Scalability & caching | Design for 100x. No N+1, paginate, index, cache, backpressure. | Redis, DB indexes |
| 21 | Cost Management | Cap paid/AI API per user. Monitor spend + alerts. | Usage logs, billing alerts |
| 22 | Testing | Unit on critical paths + integration. CI before deploy. | Jest, Pytest |
| 23 | Data Integrity & idempotency | Transactions, constraints, idempotency keys, safe (expand/contract) migrations. | ORM transactions |
| 24 | UX / Accessibility | Loading/empty/error states. Semantic HTML. WCAG 2.2 AA. | ARIA, axe |
| 25 | Maintainability | README, descriptive naming, pinned versions, modular boundaries. | README.md, lockfiles |
| 26 | Legal / Privacy | Privacy policy, deletion flow, consent, license + data-source ToS, minimization. | GDPR/CCPA |
| 27 | Operational Readiness & observability | Monitoring, metrics+traces, SLOs+alerting, backups, rollback, runbooks. | Sentry, OTel |
| 28 | **Resilience** | Timeouts on every external call, retries+backoff, circuit breakers, graceful degradation. | — |
| 29 | AI Code Reliability | Verify AI-used APIs exist + aren't deprecated. | Official docs |
| 30 | **CI/CD gates** | Lint + test + SAST + dep-scan + secret-scan block the pipeline. | Semgrep, CodeQL |
| 31 | **Threat modeling & safe rollout** | STRIDE pass before build. Feature flags, canary, rollback. | — |

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
- **Depend on interfaces, not implementations.** Abstract the swappable thing (DB, LLM provider, payment gateway, data source) behind an interface so it can change without rippling through the codebase.
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

- **AI / LLM apps.** The single most common failure is calling the provider straight from the UI with a baked-in key. Always route through a server proxy, wrap the provider in one swappable `llm` service, and treat every model response as untrusted input — schema-validate it before you act on it or render it. Apply the same to any content you fetch/scrape (indirect prompt injection, area 13).
- **MCP servers.** Think of each tool as a public API endpoint: strict input schema, least privilege, idempotent/safe where possible, and a clear error contract that never leaks internals. Keep the tool functions free of protocol code so they can be tested directly. Authorize every tool action (area 13).
- **Forward-deployed apps.** "Forward-deployed" means the app runs in *someone else's* environment, so portability and handoff beat cleverness. One configurable artifact, no phone-home without consent, and documentation an operator who has never met you can follow.

## Reference files

- `references/full-checklist.md` — the complete 31-area guide with rule bullets, code examples, and drop-in prompts. Load this whenever you need specifics beyond the summary tables above.
- `CHANGELOG.md` — version history (v2.0.0 diff + rationale).

---

Created by **Muhammad Hassaan ul Mustafa** (FAST-NUCES). Original security checklist (areas 1–13) by [@tahajaffriii](https://github.com/tahajaffriii). v2.0 production-readiness expansion: security areas 14–18, engineering areas 28/30/31, depth on 1/4/10/13, and the `references/full-checklist.md` reference.
