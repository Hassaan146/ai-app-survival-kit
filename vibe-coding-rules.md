---
name: vibe-coding-rules
description: "23-area checklist for security and engineering quality in AI-generated ('vibe coded') apps. Use whenever writing, reviewing, auditing, or shipping code for a web app, API, or backend. Covers secrets, rate limiting, input validation, auth, SQL injection, CORS, security headers, file uploads, error handling, dependencies, XSS, and AI/LLM risks (prompt injection, token costs) -- plus non-security issues: architecture, scalability, cost management, testing, data integrity, UX/accessibility, maintainability, legal/compliance, operational readiness, and AI-specific reliability pitfalls (hallucinated APIs, deprecated patterns). Trigger whenever the user asks to review code for security or quality, asks 'is this safe to ship' or 'production ready', wants a pre-deploy checklist, is building an app/API/SaaS with AI help, or mentions vibe coding, code audit, security review, or technical debt."
---



# Vibe Coding Rules — Security & Engineering Checklist

A 23-area standard for anything an AI tool generates: code, APIs, backend services, or full apps. Use it to **generate code correctly the first time**, to **review/audit existing code**, or to **run a pre-deploy gate check**.

The checklist is split into two parts:
- **Part 1 — Security (areas 1–13):** secrets, rate limiting, input validation, auth, SQL security, CORS, HTTP headers, file uploads, error handling, dependencies, XSS, deployment gate, AI/LLM-specific risks.
- **Part 2 — Engineering & Product (areas 14–23):** architecture, scalability, cost management, testing, data integrity, UX/accessibility, maintainability, legal/compliance, operational readiness, AI-specific code reliability pitfalls.

Full rule-by-rule detail — including the exact bullet checklist, code examples, and copy-paste prompts for each of the 23 areas — lives in `references/full-checklist.md`. **Read that file before doing a thorough review or generating a CLAUDE.md/.cursorrules file** — don't rely on memory of this summary for specifics.

## How to use this skill

**When generating new code (a feature, an endpoint, a full app):**
Apply the relevant rules from the Quick Reference table below proactively as you write — don't wait to be asked. At minimum: keep secrets out of frontend code, validate input server-side, use parameterized queries/ORM, rate-limit public endpoints, set security headers, and handle errors without leaking internals to the client. For anything beyond a trivial script, also apply the Part 2 basics: layered architecture, pagination on list endpoints, transactions on multi-step writes, and a README.

**When reviewing or auditing existing code:**
Open `references/full-checklist.md` and go through all 23 areas systematically against the codebase. Report findings grouped by area (Security findings first, then Engineering/Product), citing the specific rule violated and the fix. Don't just report security issues if asked for "all issues" — the user explicitly wants the non-security categories too (architecture, scalability, cost, testing, data integrity, UX, maintainability, legal, operational readiness, AI-reliability).

**When asked for a pre-deploy / "is this safe to ship" check:**
Walk through the Quick Reference table below as a gate checklist. Call out anything unmet as a blocker (security items) vs. a recommendation (engineering/product items not strictly required to ship but risky to skip).

**When asked to generate a `CLAUDE.md` or `.cursorrules` file for a project:**
Pull the "Drop-in prompt" blocks from `references/full-checklist.md` for the areas relevant to that project's stack, and assemble them into the file. Don't just summarize — use the actual prompt language from the reference file, since it's written to be pasted directly into a rules file.

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
| 12 | Deploy Gate | Run checklist before every ship. | See area 12 in reference |
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

## Reference files

- `references/full-checklist.md` — the complete 23-area guide. Each area includes: the full bullet-point rule list, a short rationale, and either a code example (security areas 1–13) or a drop-in prompt for `CLAUDE.md`/`.cursorrules` (engineering areas 14–23). Load this whenever you need the specifics, not just the summary table above.

Based on the security checklist by @tahajaffriii, expanded with engineering, product, and operational analysis.