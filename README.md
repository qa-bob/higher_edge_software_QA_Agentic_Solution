# Higher Edge Software — QA Agentic Solution

A Playwright + TypeScript regression test suite for [Higher Edge Software](https://www.highersoftware.com/), built with the **Page Object Model (POM)** and **Object-Oriented Programming (OOP)** patterns, and structured for agentic execution by [Claude Code](https://code.claude.com/docs/).

The suite exercises the live site read-only: navigation, forms, and business features are all covered, but **no form is ever submitted and no account is ever created.**

---

## Repo purpose

This repository tests one target site, configured in [`site.config.json`](./site.config.json):

| Field | Value |
|---|---|
| Site | Higher Edge Software |
| URL | https://www.highersoftware.com |
| Coverage | Smoke, navigation, forms, functional, visual, responsive |

Test coverage is organized into five categories, each tagged and runnable independently:

| Category | Tag | What it checks |
|---|---|---|
| Smoke | `@smoke` | Site loads, title present, no console errors |
| Navigation | `@navigation` | Nav links, routing, menus, breadcrumbs |
| Forms | `@forms` | Field presence and validation (never submission) |
| Functional | `@functional` | Business features — pricing, search, video, accordions, etc. |
| Visual | `@visual` | Screenshot regression |
| Responsive | `@responsive` | Layout at mobile/tablet/desktop viewports |

Together these form the **regression suite** run before any deploy of the target site.

---

## Architecture

- **Page Object Model** — every page/section is a class in `src/pages/`, extending `BasePage` (`src/pages/base.page.ts`). Locators are typed `readonly Locator` properties; methods represent user actions, never assertions.
- **OOP** — shared behavior (navigation, load waits, responsiveness checks, screenshots) lives once in `BasePage` and is inherited by every page class.
- **Fixtures** — `src/fixtures/site.fixture.ts` extends Playwright's `test` with pre-built page objects and the loaded `site.config.json`, so specs never construct page objects directly.
- **Config-driven** — `playwright.config.ts` reads the target URL from `site.config.json` (or `SITE_URL` env override). No test hardcodes a URL.

```
site.config.json          # target site + feature flags (read by playwright.config.ts)
playwright.config.ts      # desktop / mobile / tablet projects
global-setup.ts           # runs once before the suite
src/
  pages/                  # POM classes, one per page/section
  fixtures/site.fixture.ts
  utils/                  # link-checker, visual-helper
  types/
tests/
  smoke/  navigation/  forms/  functional/  visual/  responsive/
.claude/
  agents/                 # subagents — see AGENT.md
  commands/               # skills / slash commands — see SKILLS.md
  hooks/                  # lifecycle scripts, wired in .claude/settings.json
  settings.json
.github/                  # CI workflow, issue/PR templates, contributor rules
CLAUDE.md                 # instructions Claude Code reads every session
AGENT.md                  # subagent reference
SKILLS.md                 # skill/slash-command reference
```

---

## Setting up the development environment

**Prerequisites:** Node.js 18+ and npm.

```bash
# 1. Clone and install dependencies
git clone <this-repo-url>
cd higher_edge_software_QA_Agentic_Solution
npm install

# 2. Install Playwright's bundled browsers
npx playwright install

# 3. Configure environment (optional — defaults come from site.config.json)
cp .env.example .env
# Edit .env to override SITE_URL for a staging environment, e.g.:
#   SITE_URL=https://staging.highersoftware.com
```

### Running tests

```bash
npm test                    # full suite, all projects (desktop/mobile/tablet)
npm run test:smoke          # @smoke only
npm run test:navigation     # @navigation only
npm run test:forms          # @forms only
npm run test:visual         # @visual only
npm run test:responsive     # @responsive only
npm run test:headed         # any of the above with a visible browser
npm run report              # open the last HTML report
npm run baseline            # regenerate visual snapshots after an intentional UI change
```

### Code quality

```bash
npm run lint                # ESLint
npm run typecheck           # tsc --noEmit — run before every commit/PR
```

### Using Claude Code in this repo

Claude Code reads [`CLAUDE.md`](./CLAUDE.md) automatically at session start — it contains the architecture rules and tagging conventions enforced below. See [`AGENT.md`](./AGENT.md) for the subagents available in this repo and [`SKILLS.md`](./SKILLS.md) for the slash commands (`/analyze-site`, `/generate-full-suite`, `/run-smoke`, `/update-baseline`, `/generate-report`).

---

## Contributing

Read [`CLAUDE.md`](./CLAUDE.md) and [`.github/CONTRIBUTING.md`](./.github/CONTRIBUTING.md) before opening a PR. In short:

- **Tag every test** with at least one of `@smoke`, `@navigation`, `@forms`, `@functional`, `@visual`, `@responsive`.
- **Use the POM.** No raw `page.locator()` calls in spec files — add locators/methods to a page object in `src/pages/`.
- **No assertions in page objects.** `expect()` belongs in tests only.
- **Never submit a form, create an account, or log in** (unless `site.config.json`'s `auth.required` is `true` and real test credentials have been explicitly provided).
- **No hardcoded URLs.** Always resolve the target through `baseURL` / `site.config.json`.
- **No `page.waitForTimeout()`.** Use Playwright's auto-waiting or `waitForSelector`.
- **Strict TypeScript.** Avoid `any` without justification; run `npx tsc --noEmit` before pushing.
- **Open PRs against `main`** using the template in `.github/PULL_REQUEST_TEMPLATE.md`; CI (`.github/workflows/`) must pass.

---

## Related docs

- [`CLAUDE.md`](./CLAUDE.md) — instructions Claude Code loads every session (architecture rules, tagging, do-nots)
- [`AGENT.md`](./AGENT.md) — subagents defined in `.claude/agents/`
- [`SKILLS.md`](./SKILLS.md) — skills/slash commands defined in `.claude/commands/`
- [`.github/CONTRIBUTING.md`](./.github/CONTRIBUTING.md) — contributor checklist and PR process
