# Contributing

Pull requests are welcome. Here is how to contribute:

## What is in scope

- Fixes to existing rules (wrong advice, outdated tools, broken examples)
- New drop-in prompts for areas 14–23
- Additional app-type rows in the architecture table
- Clearer wording or better code examples for Part 1 security areas
- New areas that belong in a production-readiness checklist but are not covered yet

## What is out of scope

- Opinionated style changes with no correctness benefit
- Adding tools or libraries just because they are popular
- Changing the three-part structure (Security / Engineering / Architecture)

## How to contribute

1. Fork the repo.
2. Make your change in the relevant `.md` file.
3. If you add or change a rule in `references/full-checklist.md`, update the matching summary row in `vibe-coding-rules.md` too — they must stay in sync.
4. If the change would affect the PDF, note it in the PR description so the PDF can be regenerated.
5. Open a pull request with a one-line description of what changed and why.

## Attribution

If your contribution is based on an external source (blog post, checklist, research), include a link in the PR description.
