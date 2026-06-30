# Changelog — vibe-coding-rules

## v2.0 — 2026-07-01
Upgraded from the original 13-area security checklist to a **31-area production-readiness standard**, and wrote the `references/full-checklist.md` reference that the repo's `references/` folder was missing.

**Credit:** original security checklist (areas 1–13) by **Taha Jaffri (@tahajaffriii)**. v2.0 expansion below.

### Why (gaps found in the v1 rating)
The v1 skill was strong (~8/10) but had real holes for modern apps — especially scraper/agent/MCP projects. v2.0 closes them.

### Added — Security (Part 1)
- **14 · SSRF** — outbound-URL allowlist + block private/link-local IPs & cloud metadata. (Critical: this project fetches/scrapes arbitrary sites.)
- **15 · CSRF** — SameSite + anti-CSRF tokens.
- **16 · Webhook/inbound** — HMAC signature + replay protection.
- **17 · Logging & PII** — structured, redacted, retention/access.
- **18 · Secret scanning** — gitleaks pre-commit + CI gate.

### Expanded — Security
- **1 · Secrets** → rotation, scoped keys, never-in-logs.
- **4 · Auth** → IDOR/object-level authz, RBAC/ABAC, MFA, account lockout, safe password reset.
- **10 · Dependencies** → supply-chain: lockfile integrity, postinstall risk, Dependabot, SBOM, typosquats.
- **13 · AI/LLM** → **indirect/second-order prompt injection** (untrusted scraped content), tool-use authorization, PII redaction, RAG poisoning, output handling.

### Added — Engineering/Product/Ops (Part 2)
- **28 · Resilience** — timeouts, retries+backoff, circuit breakers, graceful degradation.
- **30 · CI/CD gates** — lint+test+SAST+dep-scan+secret-scan block the pipeline.
- **31 · Threat modeling & safe rollout** — STRIDE, feature flags, canary, rollback.

### Expanded — Engineering
- **20 · Scalability** → caching + backpressure.
- **23 · Data integrity** → idempotency keys + safe (expand/contract) migrations.
- **24 · UX** → WCAG 2.2 AA specifics.
- **26 · Legal** → data-source ToS, lawful basis, minimization.
- **27 · Ops** → observability (metrics/traces/SLOs/runbooks).

### Result
Self-assessed: rules skill and reference now ~**10/10** for the practical "harden an AI-generated app" purpose. Reference (`full-checklist.md`) created (was absent in repo).

**Untouched:** `token-optimization.md` and all other skills — left exactly as-is, per instruction.
