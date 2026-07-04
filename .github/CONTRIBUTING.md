# Contributing

This repo is a read-only QA regression suite for the site configured in [`site.config.json`](../site.config.json). Before opening a PR, read [`CLAUDE.md`](../CLAUDE.md) — it is the canonical rule set and is what Claude Code itself follows when generating or editing tests here.

## Setup

See the root [`README.md`](../README.md#setting-up-the-development-environment) for `npm install`, `npx playwright install`, and `.env` setup.

## Rules (enforced in review)

- **Tag every test** with at least one of `@smoke`, `@navigation`, `@forms`, `@functional`, `@visual`, `@responsive`.
- **Page Object Model only.** No `page.locator()` calls inside `tests/**`; add locators and action methods to a class in `src/pages/` instead.
- **No assertions in page objects.** `expect()` belongs in spec files.
- **Never submit a form, sign up, or log in** — this suite must not create real records on the target site. Exception: `site.config.json` sets `auth.required: true` and explicit test credentials have been provided.
- **No hardcoded URLs.** Resolve the target through Playwright's `baseURL` (sourced from `site.config.json` / `SITE_URL`).
- **No `page.waitForTimeout()`.** Use `waitForSelector` or Playwright's auto-waiting.
- **Strict TypeScript.** No implicit `any`; run `npx tsc --noEmit` before pushing.

## Before opening a PR

```bash
npm run lint
npm run typecheck
npm test
```

All three must pass locally. CI (`.github/workflows/playwright.yml`) re-runs them on every PR against `main`.

## Adding coverage for a new page or feature

1. Add a page object class in `src/pages/<name>.page.ts`, extending `BasePage`.
2. Register it as a fixture in `src/fixtures/site.fixture.ts` if tests will need it.
3. Add a spec file to the matching `tests/<category>/` directory, tagged appropriately.
4. If the feature is unique to this site (not something the generic suite would cover), see the `test-generator` subagent in [`AGENT.md`](../AGENT.md).

## Visual baselines

Screenshots under `__snapshots__/` are the source of truth for `@visual` tests. After an intentional design change, run `npm run baseline` (or `/update-baseline`) and **visually review every updated PNG** before committing — never commit a baseline update you haven't looked at.
