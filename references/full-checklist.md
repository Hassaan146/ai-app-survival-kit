# Full Checklist — Complete 23-Area Guide (+ Part 3)

The complete companion to `vibe-coding-rules.md`. Each area has: the **rules** (the
bullet checklist), a short **why**, and either a **code example** (security areas
1–13) or a **drop-in prompt** you can paste into a `CLAUDE.md` / `.cursorrules`
file (engineering areas 14–23 and Part 3).

Use it three ways: generate code correctly the first time, audit existing code
area-by-area, or run a pre-deploy gate. Security items are **blockers**;
engineering/product/architecture items are **strong recommendations**.

---

# Part 1 — Security (areas 1–13)

## 1. Secrets

**Rules**
- All secrets (API keys, DB credentials, tokens, signing keys) live in environment variables loaded from `.env`. Never hardcoded; never in client/frontend bundles.
- `.env` is git-ignored; commit a `.env.example` with key *names* and dummy values only.
- Frontend-exposed vars (`VITE_*`, `NEXT_PUBLIC_*`) are public — never put a secret behind one.
- Treat any secret ever committed to git as compromised: rotate it immediately.

**Why** — Anything in a client bundle or git history is effectively public.

```js
// .env  (git-ignored)         .env.example  (committed)
// STRIPE_KEY=sk_live_xxx       STRIPE_KEY=sk_test_replace_me
import 'dotenv/config';
const stripe = new Stripe(process.env.STRIPE_KEY); // server-side only
```

## 2. Rate Limiting

**Rules**
- Auth endpoints (login / signup / reset): ~5 requests / 15 min per IP + account.
- General API: ~60 requests / min per IP or user; stricter on expensive routes.
- Return `429` with a `Retry-After` header. Use a shared store (Redis) across instances.

**Why** — Stops brute-force, credential stuffing, scraping, and cost-bombing.

```js
import rateLimit from 'express-rate-limit';
app.use('/auth', rateLimit({ windowMs: 15 * 60 * 1000, max: 5 }));
app.use('/api', rateLimit({ windowMs: 60 * 1000, max: 60 }));
```

## 3. Input Validation

**Rules**
- Validate **all** input server-side with a schema — body, query, params, headers, file metadata. Client validation is UX only.
- Whitelist allowed fields and reject unknown ones; check types, lengths, ranges, formats.

**Why** — The client is attacker-controlled; the server is the only trust boundary.

```js
import { z } from 'zod';
const schema = z.object({ email: z.string().email(), age: z.number().int().min(0).max(120) });
const data = schema.parse(req.body); // throws on invalid → 400
```

## 4. Auth

**Rules**
- Hash passwords with bcrypt (cost ≥ 12) or argon2 — never plaintext, never fast hashes (MD5/SHA1).
- Short-lived access tokens (~15 min) + refresh tokens. Store tokens in `httpOnly`, `Secure`, `SameSite` cookies — **not** `localStorage`.
- Enforce authorization on every protected route server-side; hiding UI is not security.
- Slow/lock repeated failures; verify email; offer MFA on sensitive actions.

**Why** — Auth is the front door; weak hashing or client-stored tokens hand it over.

```js
const hash = await bcrypt.hash(password, 12);
res.cookie('token', jwt, { httpOnly: true, secure: true, sameSite: 'lax', maxAge: 900_000 });
```

## 5. SQL Security

**Rules**
- Parameterized queries or an ORM **only**. Never concatenate user input into SQL.
- Use a least-privilege DB user. Never build table/column names from raw input.

**Why** — String-built SQL is the classic injection hole.

```js
// BAD:  `SELECT * FROM users WHERE id = ${id}`
db.query('SELECT * FROM users WHERE id = $1', [id]); // parameterized
```

## 6. CORS

**Rules**
- No `*` in production. Explicit origin whitelist; allow only required methods/headers.
- `credentials: true` only with a specific origin, never with `*`.

**Why** — A wildcard CORS policy lets any site call your API with the user's session.

```js
app.use(cors({ origin: ['https://app.example.com'], credentials: true }));
```

## 7. HTTP Headers

**Rules**
- Set CSP (restrict script/style/connect sources), HSTS, `X-Frame-Options: DENY`, `X-Content-Type-Options: nosniff`, a sane `Referrer-Policy`.
- Use `helmet` (Node) / `django-csp` / framework equivalent; configure CSP explicitly.

**Why** — Defense in depth against XSS, clickjacking, MIME sniffing, downgrade attacks.

```js
import helmet from 'helmet';
app.use(helmet());
```

## 8. File Uploads

**Rules**
- Validate MIME type **and** extension **and** magic bytes; whitelist allowed types. Cap size.
- Rename to a UUID, strip the path, store outside the web root or in object storage (S3) with no execute permission.

**Why** — Unchecked uploads enable web shells, path traversal, and storage abuse.

```js
const upload = multer({
  limits: { fileSize: 5_000_000 },
  fileFilter: (req, file, cb) => cb(null, ALLOWED.includes(file.mimetype)),
});
```

## 9. Error Handling

**Rules**
- Generic message to the client (`"Something went wrong"`); full context (stack, inputs) to server logs only.
- Never leak stack traces, SQL, file paths, or secrets to the client. Centralize in error middleware; alert on errors.

**Why** — Verbose errors are a free recon map for attackers.

```js
app.use((err, req, res, next) => {
  logger.error({ err, path: req.path });
  res.status(500).json({ error: 'Internal error' });
});
```

## 10. Dependencies

**Rules**
- Run `npm audit` / `pip-audit` after every install and in CI; fix high/critical.
- Commit lockfiles and pin versions in production. Avoid unmaintained or low-trust packages; review transitive deps. Automate updates (Dependabot/Renovate).

**Why** — Most real-world breaches ride in through a vulnerable dependency.

```bash
npm audit --audit-level=high   # fail CI on high+ severity
```

## 11. XSS

**Rules**
- No `dangerouslySetInnerHTML` / `v-html` / `innerHTML` with user data. If unavoidable, sanitize with DOMPurify.
- No `eval`, `new Function`, inline event handlers, or HTML built by string concat. Rely on framework escaping; add CSP as backstop.

**Why** — Injected script runs with the victim's session.

```js
import DOMPurify from 'dompurify';
el.innerHTML = DOMPurify.sanitize(userHtml);
```

## 12. Deploy Gate

**Rules** — Before every ship, confirm: secrets out of code · prod env vars set · HTTPS + HSTS · security headers · CORS locked · rate limits on · server-side validation on · `audit` clean · generic error handling · backups + rollback ready · monitoring on. Any unmet **security** item blocks the deploy.

**Why** — A checklist run on every ship catches the one thing you forgot this time.

## 13. AI / LLM Security

**Rules**
- Provider API keys server-side only; the browser calls **your** proxy, never the model API.
- Treat user input *and* retrieved content as untrusted: guard against prompt injection (don't let it override system instructions; constrain tool/function use; isolate untrusted text).
- Cap `max_tokens`, set per-user/session budgets, add timeouts + retries with backoff.
- Validate/parse model output against a schema before acting on or rendering it. Never `eval` model output.

**Why** — LLM apps add two new attack surfaces: leaked keys and injected instructions, plus runaway cost.

```js
const r = await openai.chat.completions.create({ model: 'gpt-4o', max_tokens: 500, messages });
const out = OutputSchema.parse(JSON.parse(r.choices[0].message.content)); // validate before use
```

---

# Part 2 — Engineering & Product (areas 14–23)

Each area below ships with a **drop-in prompt** — paste it into `CLAUDE.md` / `.cursorrules` so the AI applies the rule proactively.

## 14. Architecture

**Rules** — Layered structure (presentation / logic / data). One pattern per concern. Dependencies point inward. Thin controllers, fat services, repositories for data access. No business logic in routes or UI.

**Drop-in prompt**
```
Use a layered architecture: keep transport (HTTP/UI), business logic, and data
access in separate layers. Business logic must not import transport or UI code.
Controllers/handlers stay thin and delegate to services; data access goes through
repositories. Pick one pattern per concern (state, data-fetching, errors) and use
it consistently. Expose small public interfaces per module; keep internals private.
```

## 15. Scalability

**Rules** — Design for 100× current data. Paginate every list endpoint. Index columns you filter/sort on. Kill N+1 queries (eager-load/batch). Cache hot reads (Redis). Move slow work to async jobs/queues. Make services stateless/horizontally scalable.

**Drop-in prompt**
```
Assume data and traffic grow 100x. Paginate all list endpoints (cursor or
limit/offset). Add DB indexes for every filtered/sorted/joined column. Avoid N+1
queries — batch or eager-load. Cache expensive reads and invalidate deliberately.
Offload slow/external work to a queue. Keep request handlers stateless so the
service scales horizontally.
```

## 16. Cost Management

**Rules** — Cap paid-API usage per user/tenant. Track spend with usage logs and billing alerts. Cache and batch external/LLM calls. Set hard ceilings (token/request budgets) that fail closed. Prefer cheaper models/tiers where adequate.

**Drop-in prompt**
```
Treat every paid API/LLM call as a cost. Enforce per-user and global usage caps
that fail closed when exceeded. Log usage and set billing alerts. Cache and batch
calls where possible, add retries with backoff (not infinite). Choose the cheapest
model/tier that meets the quality bar; make the model/provider configurable.
```

## 17. Testing

**Rules** — Unit-test critical/pure logic (validation, pricing, auth rules). Integration-test key flows (signup, checkout, the core feature). Run tests in CI as a deploy gate. Cover edge cases and failure paths, not just the happy path.

**Drop-in prompt**
```
Write unit tests for all critical and pure logic (validation, calculations, auth
rules) and integration tests for the core user flows. Cover edge cases and error
paths, not just the happy path. Tests must run in CI and block deploy on failure.
Keep tests fast and deterministic; mock external services.
```

## 18. Data Integrity

**Rules** — Wrap multi-step writes in transactions (all-or-nothing). Enforce constraints in the DB (foreign keys, unique, not-null, checks) — not just in app code. Make mutating operations idempotent where retried. Use migrations for schema changes; never edit prod schema by hand.

**Drop-in prompt**
```
Wrap any operation that writes to more than one row/table in a transaction so it
is all-or-nothing. Enforce invariants with database constraints (FK, unique,
not-null, check) in addition to app-level checks. Make retry-able mutations
idempotent. Manage all schema changes through versioned migrations.
```

## 19. UX / Accessibility

**Rules** — Every async view has loading, empty, and error states. Use semantic HTML (`button`, `nav`, `main`, headings in order). Keyboard-navigable with visible focus states. Meet WCAG AA contrast. Label inputs; use ARIA only where semantics fall short. Respect `prefers-reduced-motion`.

**Drop-in prompt**
```
Every screen that loads data must handle loading, empty, and error states
explicitly. Use semantic HTML and a correct heading order. Ensure full keyboard
navigation with visible :focus-visible states and WCAG AA contrast. Label all form
inputs and associate errors with their field. Add ARIA only where native semantics
are insufficient. Respect prefers-reduced-motion.
```

## 20. Maintainability

**Rules** — README with run/scripts/env/architecture. Descriptive names; small single-purpose functions. Pin versions + commit lockfile. Consistent formatting/lint (Prettier + ESLint, bridged by eslint-config-prettier). No dead code, no commented-out blocks, necessary comments only.

**Drop-in prompt**
```
Keep the codebase maintainable: a README (what/run/scripts/env/architecture),
descriptive names, and small single-purpose functions. Pin dependency versions and
commit the lockfile. Enforce one formatter (Prettier) and one linter (ESLint), with
eslint-config-prettier so they don't conflict. No dead or commented-out code.
Comments explain non-obvious "why", never the obvious "what".
```

## 21. Legal / Privacy

**Rules** — Publish a privacy policy and terms. Provide data export and deletion (GDPR/CCPA). Cookie/consent banner where required. Minimize and encrypt PII; define retention. Check dependency and asset licenses (fonts, images, code).

**Drop-in prompt**
```
Add a privacy policy and terms of service. Implement user data export and deletion
flows (GDPR/CCPA). Collect only the PII you need, encrypt it at rest/in transit,
and set a retention period. Add consent where required (cookies/analytics). Verify
licenses for all dependencies and assets (fonts, images, sample code).
```

## 22. Operational Readiness

**Rules** — Monitoring + alerting (errors, latency, uptime). Structured, searchable logs (no secrets/PII). Health-check endpoint. Automated backups with a **tested** restore. A rollback plan and a runbook before launch.

**Drop-in prompt**
```
Before launch, wire up monitoring and alerting for errors, latency, and uptime.
Emit structured logs (no secrets/PII). Expose a health-check endpoint. Configure
automated backups and actually test a restore. Document a rollback plan and an
operations runbook (deploy, common incidents, on-call).
```

## 23. AI Code Reliability

**Rules** — Verify any AI-suggested API/method/flag actually exists in the official docs before relying on it. Watch for deprecated patterns and out-of-date library usage. Pin model and SDK versions. Don't trust hallucinated config keys, env vars, or imports. Run and test generated code, don't assume it works.

**Drop-in prompt**
```
Do not assume an AI-suggested API, method, flag, or config key exists — verify it
against the official, current docs before using it. Avoid deprecated patterns; use
the version that matches the installed package. Pin model IDs and SDK versions.
Run and test generated code before trusting it; treat "it compiles" as not enough.
```

---

# Part 3 — Architecture, Modularity & Documentation

Applies to **every** app, on top of the per-area rules above. Part 1 and 2 tell you
what not to ship; Part 3 shapes the codebase so it stays reviewable, reusable, and
handoff-ready.

## The universal standard

**Architecture**
- Layered separation (presentation / logic / data); transport stays at the edge.
- Dependencies point inward (ports & adapters); the core is framework-agnostic.
- One pattern per concern; explicit module boundaries with small public interfaces.
- Config via environment, never hardcoded (twelve-factor).

**Modularity & reusability**
- Single responsibility — nameable in one phrase; if you need "and", split it.
- Extract on the second occurrence; never copy-paste logic.
- Build composable primitives (components, hooks, repositories, middlewares) and compose features from them.
- Depend on interfaces, not implementations, so swappable pieces (DB, LLM, payments) can change without rippling.
- No god-files; co-locate related files; use barrel/`index` exports.
- Separate data from presentation and logic from view.

**Documentation**
- README is mandatory (what / run / scripts / env / architecture overview).
- Necessary comments only — non-obvious *why*, never the obvious *what*.
- Doc headers on modules/exports (responsibility + inputs/outputs).
- `ARCHITECTURE.md` for non-trivial apps (pipeline, "where do I change X" map, boundaries).
- Update docs in the **same commit** as the code — stale docs are worse than none.

**Drop-in prompt (universal)**
```
Shape every app for reuse and handoff. Use a layered architecture with
dependencies pointing inward and one pattern per concern. Keep modules small and
single-purpose; extract shared logic the second time it appears; build composable
primitives and compose features from them. Depend on interfaces for swappable
pieces (DB, LLM provider, payments). No god-files; use barrels and co-location.
Separate data from presentation and logic from view. Drive config from the
environment. Ship a README, doc headers, and (for non-trivial apps) an
ARCHITECTURE.md, and update docs in the same commit as the code. Comments explain
why, not what.
```

## Correct architecture by app type

Apply the matching layering on top of the universal standard.

| App type | Layering (edge → core) | Must-haves |
|---|---|---|
| **Frontend-only** | routing → pages → feature components → reusable UI primitives → hooks (logic) → services (API client) → utils | Data/content separated from views; logic in hooks; one design-token source; route-level code-splitting; error boundary; client validation is UX only (if a backend exists, it re-validates server-side) |
| **Backend-only** | routes/controllers → services → repositories → models | Thin controllers, fat services; validate every input at the boundary; ORM/parameterized only; cross-cutting concerns as middleware; pagination + transactions + indexes |
| **Full-stack** | frontend layers ⟂ backend layers, joined by a typed contract | One source of truth for the API shape (shared types / OpenAPI / schema pkg); shared code in a real boundary, not copy-paste; independently deployable units; auth + CORS + server-side validation end-to-end |
| **AI / LLM app** | UI → app logic → **LLM service (provider behind an interface)** → provider SDK | Keys server-side only (browser hits your proxy); model output is untrusted (parse/validate/sanitize); token & cost caps; prompts versioned as assets + evals; pin model IDs; verify the SDK method exists |
| **MCP server** | transport (stdio/HTTP) → tool registry → **individual tools** → domain logic | Each tool = one small, well-named function with a strict input schema and clear output contract; validate inputs (untrusted); least privilege; transport separate from tool logic (unit-testable); document purpose/args/side-effects; confirm destructive actions |
| **Forward-deployed app** (on-prem / per-client / edge) | same artifact across clients; **differences only in config** | Configuration-first (zero hardcoded client specifics); reproducible builds + pinned deps; self-contained & offline-tolerant; handoff docs + ops runbook (install/upgrade/rollback); secrets via the client's store; backups + rollback before launch |

**Per-type notes**
- **AI / LLM apps** — The #1 failure is calling the provider straight from the UI with a baked-in key. Route through a server proxy, wrap the provider in one swappable `llm` service, and schema-validate every response before acting on it or rendering it.
- **MCP servers** — Treat each tool like a public API endpoint: strict input schema, least privilege, idempotent/safe where possible, and a clear error contract that never leaks internals. Keep tool functions free of protocol code so they're directly testable.
- **Forward-deployed apps** — "Forward-deployed" means the app runs in *someone else's* environment, so portability and handoff beat cleverness. One configurable artifact, no phone-home without consent, and docs an operator who has never met you can follow.

## Part 3 gate

Before "done": layered separation holds · no god-files / duplicated logic ·
interfaces abstract the swappable pieces · config is env-driven · README +
(non-trivial) ARCHITECTURE.md exist and match the code · comments explain *why* ·
the per-app-type must-haves are met.

---

Based on the security checklist by @tahajaffriii, expanded with engineering,
product, operational, and architecture analysis.
