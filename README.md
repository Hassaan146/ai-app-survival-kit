# AI App Survival Kit

[![GitHub stars](https://img.shields.io/github/stars/Hassaan146/ai-app-survival-kit?style=flat)](https://github.com/Hassaan146/ai-app-survival-kit/stargazers)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)

**Created by Muhammad Hassaan ul Mustafa — FAST-NUCES (National University of Computer and Emerging Sciences).**

A practical starter kit for building AI-generated apps with fewer blind spots, lower token waste, and a stronger production-readiness checklist.

This repository includes two Claude Skills plus a PDF version of the app quality checklist:

- `token-optimization.md` — switches the AI into an ultra-compressed communication style, cutting token usage by roughly 75% without losing technical accuracy.
- `vibe-coding-rules.md` — a 31-area checklist for security, engineering quality, product readiness, architecture/modularity, and AI-specific risks.
- `references/full-checklist.md` — the full rule-by-rule detail (code examples + drop-in prompts) that `vibe-coding-rules.md` references.
- `Complete Rules for AI-Generated Apps.pdf` — a shareable snapshot of the checklist. **Note:** the PDF is manually generated and may lag behind the markdown. The `.md` files are always the source of truth.

## Contents

- [Quick Start](#quick-start)
- [Token Optimization Skill](#token-optimization-skill)
- [Vibe Coding Rules Skill](#vibe-coding-rules-skill)
- [Files](#files)
- [Why This Exists](#why-this-exists)
- [Author](#author)
- [Credits](#credits)
- [License](#license)

## Quick Start

### Claude Code (recommended)

1. Copy `vibe-coding-rules.md` and `token-optimization.md` into your project's `.claude/skills/` folder (create it if it does not exist).
2. Copy the entire `references/` folder into `.claude/skills/references/` so the full checklist is reachable.
3. In Claude Code, type `/vibe-coding-rules` to trigger the checklist skill, or `/caveman` to activate token-compression mode.

### Cursor / Windsurf / any `.cursorrules`-based tool

1. Open `references/full-checklist.md`.
2. Copy the **Drop-in prompt** blocks for the areas relevant to your project's stack.
3. Paste them into your `.cursorrules` or `CLAUDE.md` file.
4. The universal drop-in prompt at the end of Part 3 is a good starting point for any project.

### Manual / team use

Share `Complete Rules for AI-Generated Apps.pdf` for review sessions, onboarding, or pre-deploy gate checks. For the most current version, print from `references/full-checklist.md`.

## Token Optimization Skill

The token optimization skill is designed for moments when you want an AI coding assistant to stay technically accurate while using far fewer tokens.

It works by switching the assistant into an ultra-compressed communication style:

- Removes filler words, pleasantries, hedging, and repeated explanations.
- Drops unnecessary articles and long phrasing where meaning is still clear.
- Keeps technical terms, code blocks, exact errors, commands, and important warnings intact.
- Uses compact patterns like: `[thing] [action] [reason]. [next step].`
- Supports multiple compression levels: `lite`, `full`, `ultra`, `wenyan-lite`, `wenyan-full`, and `wenyan-ultra`.
- Automatically returns to clearer language for security warnings, irreversible actions, or complex steps where compression could cause confusion.

In short: fluff gets removed, technical substance stays.

## Vibe Coding Rules Skill

The second skill is a 31-point checklist for the things AI coding tools often skip when generating apps, APIs, and backend systems.

Instead of manually feeding these rules into an AI coding tool every time, the checklist is packaged as a Claude Skill. Attach it once, and every app you build can inherit the same security and engineering standard automatically.

### Security: 18 Areas

1. Secrets management & rotation
2. Rate limiting
3. Input validation
4. Authentication & authorization (RBAC/IDOR, MFA)
5. SQL injection prevention
6. CORS configuration
7. Security headers
8. File uploads
9. Error handling
10. Dependency & supply-chain audits
11. XSS prevention
12. Deployment gate checks
13. AI/LLM-specific risks (direct + indirect prompt injection, tool-use authz, PII redaction, token-cost attacks)
14. SSRF (server-side request forgery)
15. CSRF
16. Webhook/inbound integration security
17. Logging & PII handling
18. Secret scanning

### Engineering & Product: 13 Areas

1. Architecture
2. Scalability & caching
3. Cost management
4. Testing
5. Data integrity & idempotency
6. UX and accessibility (WCAG 2.2 AA)
7. Maintainability
8. Legal and compliance readiness
9. Operational readiness & observability
10. AI-specific blind spots (hallucinated APIs, deprecated patterns)
11. Resilience (timeouts, retries, circuit breakers, graceful degradation)
12. CI/CD gates (lint, test, SAST, dependency + secret scanning)
13. Threat modeling & safe rollout (STRIDE, feature flags, canary, rollback)

## Files

| File | Purpose |
| --- | --- |
| `token-optimization.md` | Token compression skill for shorter, cheaper, high-signal AI responses |
| `vibe-coding-rules.md` | 31-area production-readiness skill for AI-generated apps |
| `references/full-checklist.md` | Full rule detail, code examples, and drop-in prompts for each area |
| `Complete Rules for AI-Generated Apps.pdf` | Shareable snapshot of the checklist (may lag behind the markdown) |

## Why This Exists

AI coding tools move fast, but they can quietly skip boring-but-critical production work: auth hardening, input validation, cost limits, tests, monitoring, accessibility, compliance, and API verification.

This kit turns those checks into reusable workflow assets so every new AI-built app starts with better defaults.

## Author

**Muhammad Hassaan ul Mustafa**
FAST-NUCES (National University of Computer and Emerging Sciences)
GitHub: [@Hassaan146](https://github.com/Hassaan146)

## Credits

The security checklist (areas 1–13) is based on the original checklist by [@tahajaffriii](https://github.com/tahajaffriii), expanded, restructured, and extended with engineering, product, operational, and architecture analysis by Muhammad Hassaan ul Mustafa.

## License

MIT © 2026 Muhammad Hassaan ul Mustafa — see [LICENSE](LICENSE).
