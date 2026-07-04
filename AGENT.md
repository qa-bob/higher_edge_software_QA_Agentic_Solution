# Agents in this repo

This project uses [Claude Code subagents](https://code.claude.com/docs/en/sub-agents) — specialized assistants with their own system prompt, tool access, and context window — to automate the parts of QA work that are repetitive and would otherwise flood the main conversation (crawling pages, comparing DOM structure, generating boilerplate specs).

Subagent files live in `.claude/agents/` and are Markdown with required YAML frontmatter:

```markdown
---
name: kebab-case-unique-id
description: When Claude should delegate to this subagent
tools: Read, Write, Grep        # optional — inherits everything if omitted
model: inherit                  # optional — sonnet | opus | haiku | fable | inherit
---

System prompt body — what the subagent does, step by step.
```

`name` and `description` are the only required fields. Claude Code matches new tasks against every subagent's `description`, so that field should say precisely when to delegate to it.

---

## Subagents defined here

### `site-analyzer` (`.claude/agents/site-analyzer.md`)

Crawls a live site and produces a fully-populated `site.config.json`.

- **Invoke when:** onboarding a new target site, re-verifying `site.config.json` after a redesign, or running `/analyze-site`.
- **Tools:** `WebFetch`, `Read`, `Write`, `Bash`.
- **Handles:** SPA hydration delays, cookie-consent banners, auth-gated sites, redirect chains, sites with no visible nav.
- **Output:** a `site.config.json` block, an issues checklist, and a confidence rating (High/Medium/Low).

### `test-generator` (`.claude/agents/test-generator.md`)

Reads a populated `site.config.json` and writes site-specific Playwright specs that go beyond the shared suites, landing in `tests/custom/`.

- **Invoke when:** a discovered feature needs bespoke coverage, generic selectors don't fit the site's structure, or you're writing a regression test for a known bug.
- **Tools:** `Read`, `Write`, `Edit`, `Glob`, `Grep`, `WebFetch`.
- **Conventions enforced:** imports from `@fixtures/site.fixture`, tags every test (`@custom` plus a category tag), strict TypeScript, no fixed timeouts over 500ms, never submits a form.

Both agents are **project-scoped** (`.claude/agents/`, not `~/.claude/agents/`), so they're versioned with the repo and available to every contributor who checks it out.

---

## Adding a new subagent

1. Create `.claude/agents/<name>.md` with the frontmatter shown above. Keep `name` lowercase-with-hyphens and unique across the tree.
2. Write the system prompt as the file body: role, when to invoke, step-by-step instructions, edge cases, expected output format. Follow the existing two files as a template.
3. Restrict `tools` to the minimum the agent needs — a site-crawling agent doesn't need `Edit`, for example.
4. Test it by asking Claude Code in a session: *"Use the `<name>` agent to …"*. If it isn't picked up, restart the session (a newly created `.claude/agents/` directory isn't detected mid-session).
5. Commit the file — subagents in `.claude/agents/` are shared with the whole team via source control.

## Agents vs. skills

Subagents (this file) run in an **isolated context window** with their own tool restrictions — use them for research/generation work you don't want cluttering the main conversation. [Skills](./SKILLS.md) are lighter-weight procedures that run inline in the current conversation and are invoked with `/name`. This repo's `/analyze-site` and `/generate-full-suite` skills are the user-facing entry points; they can delegate to the `site-analyzer` and `test-generator` subagents internally when the task benefits from a separate context.
