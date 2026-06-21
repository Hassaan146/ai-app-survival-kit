# AI App Survival Kit

A practical starter kit for building AI-generated apps with fewer blind spots, lower token waste, and a stronger production-readiness checklist.

This repository includes two Claude Skills plus a PDF version of the app quality checklist:

- `token-optimization.md`: a token optimization skill that can reduce conversational token usage by about 75%.
- `vibe-coding-rules.md`: a 23-area checklist for security, engineering quality, product readiness, and AI-specific risks.
- `Complete Rules for AI-Generated Apps.pdf`: a shareable PDF version of the AI-generated app checklist.

## Token Optimization Skill

The token optimization skill is designed for moments when you want an AI coding assistant to stay technically accurate while using far fewer tokens.

It works by switching the assistant into an ultra-compressed communication style:

- Removes filler words, pleasantries, hedging, and repeated explanations.
- Drops unnecessary articles and long phrasing where meaning is still clear.
- Keeps technical terms, code blocks, exact errors, commands, and important warnings intact.
- Uses compact patterns like: `[thing] [action] [reason]. [next step].`
- Supports multiple compression levels: `lite`, `full`, `ultra`, `wenyan-lite`, `wenyan-full`, and `wenyan-ultra`.
- Automatically returns to clearer language for security warnings, irreversible actions, or complex steps where compression could cause confusion.

In short: fluff gets removed, technical substance stays. That is how the skill can cut token usage by roughly 75% without losing the actual engineering value.

## Vibe Coding Rules Skill

The second skill is a 23-point checklist for the things AI coding tools often skip when generating apps, APIs, and backend systems.

Instead of manually feeding these rules into an AI coding tool every time, the checklist is packaged as a Claude Skill. Attach it once, and every app you build can inherit the same security and engineering standard automatically.

### Security: 13 Areas

The security checklist covers:

1. Secrets management
2. Rate limiting
3. Input validation
4. Authentication
5. SQL injection prevention
6. CORS configuration
7. Security headers
8. File uploads
9. Error handling
10. Dependency audits
11. XSS prevention
12. Deployment gate checks
13. AI/LLM-specific risks, including prompt injection and token-cost attacks

### Engineering & Product: 10 Areas

The engineering and product checklist covers:

1. Architecture
2. Scalability
3. Cost management
4. Testing
5. Data integrity
6. UX and accessibility
7. Maintainability
8. Legal and compliance readiness
9. Operational readiness
10. AI-specific blind spots, including hallucinated APIs and deprecated patterns

## How To Use

1. Add the relevant skill file to your Claude Skills setup.
2. Enable the skill when building, reviewing, or preparing to deploy an AI-generated app.
3. Use the PDF when you want a portable checklist to review manually or share with a team.
4. Reuse the skills across projects so security, cost, quality, and reliability checks are applied by default.

## Files

| File | Purpose |
| --- | --- |
| `token-optimization.md` | Token compression skill for shorter, cheaper, high-signal AI responses |
| `vibe-coding-rules.md` | 23-area production-readiness skill for AI-generated apps |
| `Complete Rules for AI-Generated Apps.pdf` | PDF version of the checklist |

## Why This Exists

AI coding tools move fast, but they can quietly skip boring-but-critical production work: auth hardening, input validation, cost limits, tests, monitoring, accessibility, compliance, and API verification.

This kit turns those checks into reusable workflow assets so every new AI-built app starts with better defaults.
