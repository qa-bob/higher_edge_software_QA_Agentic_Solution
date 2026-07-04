# Skills in this repo

[Skills](https://code.claude.com/docs/en/skills) package a repeatable procedure so it can be invoked with `/name` instead of re-explaining it every session. This repo's skills live as flat files in `.claude/commands/` — the original "custom command" format. A file at `.claude/commands/deploy.md` and a directory-based skill at `.claude/skills/deploy/SKILL.md` both register `/deploy` and behave the same way; files under `commands/` need no YAML frontmatter at all (Claude Code derives the command name from the filename). Frontmatter only becomes useful if a skill needs `allowed-tools`, `disable-model-invocation`, argument hints, or a forked-subagent context — none of the skills below need that yet.

---

## Skills defined here

| Command | File | Purpose |
|---|---|---|
| `/analyze-site` | `.claude/commands/analyze-site.md` | Inspect the live site and produce a fully-populated `site.config.json` plus an issues checklist. |
| `/generate-full-suite` | `.claude/commands/generate-full-suite.md` | Analyze the site end-to-end and generate the complete POM (`src/pages/`) and test suite (`tests/`) across all six categories. |
| `/run-smoke` | `.claude/commands/run-smoke.md` | Run `npm run test:smoke` and print a pass/fail table with suggested fixes for common failure patterns. |
| `/update-baseline` | `.claude/commands/update-baseline.md` | Run `npm run baseline`, list which snapshots changed, and remind the reviewer to visually confirm before committing. |
| `/generate-report` | `.claude/commands/generate-report.md` | Summarize `test-results/results.json` into a per-tag pass/fail/flaky table and open the HTML report. |

### How `/analyze-site` and `/generate-full-suite` relate to the agents

Both skills describe multi-step procedures that a plain conversation can follow directly. When the site-crawling or file-generation work is heavy enough to want its own context window, Claude Code may delegate the relevant step to the `site-analyzer` or `test-generator` subagent described in [`AGENT.md`](./AGENT.md) — the skill is the entry point you invoke; the subagent is an implementation detail that keeps the main conversation uncluttered.

---

## Adding a new skill

1. **Quick command, no supporting files:** add `.claude/commands/<name>.md`. No frontmatter required — just describe the procedure. This is the right choice for everything currently in this repo.
2. **Needs bundled scripts/reference docs or invocation control:** create `.claude/skills/<name>/SKILL.md` instead, with frontmatter such as:

   ```yaml
   ---
   name: my-skill
   description: What it does and when to use it — Claude uses this to decide when to auto-invoke it
   disable-model-invocation: true   # set true for anything with side effects (deploys, commits, etc.)
   allowed-tools: Bash Read Write
   ---
   ```

   Put large reference material in sibling files (`reference.md`, `examples.md`) next to `SKILL.md` — they only load into context when actually needed, unlike the main body.
3. Tag the skill file with the project's `@tag` conventions if it generates or runs tests, so its output stays consistent with `tests/`.
4. Commit it under `.claude/commands/` or `.claude/skills/` — both are project-scoped and shared with the team via source control.

## Skills vs. CLAUDE.md vs. agents

- **[`CLAUDE.md`](./CLAUDE.md)** — facts and rules loaded into *every* session (architecture, tagging, do-nots). Use it for things Claude should always know.
- **Skills (this file)** — multi-step procedures loaded *on demand* when invoked with `/name` or when Claude judges them relevant. Use them when a CLAUDE.md section has grown into "first do X, then Y, then Z."
- **[Subagents](./AGENT.md)** — isolated-context workers a skill (or Claude) can delegate to for research- or generation-heavy sub-tasks.
