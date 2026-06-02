# Invoice Processing — sample app with the `verify` skill pre-wired

A small React + Vite invoice-processing app shipped together with the **`verify`** Copilot CLI skill, so you can see the skill drive a full QA pass on a real app end-to-end.

For the skill on its own (without this app), see the sibling [`../skill/`](../skill/) folder.

## What's here

| Part | Where |
|------|-------|
| **Demo app source** | [`src/`](./src/), [`api/`](./api/) — React + Vite frontend, Azure SWA + Functions backend |
| **Existing Playwright tests** | [`tests/`](./tests/) — what the verify skill heals + extends |
| **The `verify` skill** | [`.github/skills/verify/`](./.github/skills/verify/) — Copilot CLI auto-detects this when run from this folder |
| **App spec** | [`PRD.md`](./PRD.md) — what the app does, screen by screen |
| **Sample skill outputs** | [`testing-skills.md`](./testing-skills.md), [`report.md`](./report.md), [`skills-comparison.md`](./skills-comparison.md) — what the skill produces on this app |
| **Screenshots** | [`screenshots/`](./screenshots/) — visual reference |

## Run the demo

### 1. Install + start the app

```bash
npm install
npm run dev
# vite serves at http://localhost:5001
```

### 2. Run the existing Playwright tests (sanity check)

```bash
npx playwright test
```

### 3. Try the `verify` skill on it

Prerequisites:

- **Azure subscription** with permissions to create Playwright Workspaces — [quickstart](https://learn.microsoft.com/en-us/azure/app-testing/playwright-workspaces/quickstart-run-end-to-end-tests?tabs=playwrightcli&pivots=playwright-test-runner)
- **Playwright Workspace resource ID** — get one from the Azure portal
- **Azure CLI** signed in (`az login`)
- **Copilot CLI** installed and signed in

Then, from this folder:

```bash
copilot
# then: /verify
```

Copilot CLI picks up `.github/skills/verify/` automatically — no plugin install, no manual registration. The skill drives a real browser via Playwright Workspaces, verifies the app's flows, authors new tests for any gaps, classifies and heals failing tests, and produces a consolidated report.

## Sample outputs

For an example of what the skill produces on this app (without running it yourself), see:

- [`testing-skills.md`](./testing-skills.md) — hand-written verification report from a real run
- [`report.md`](./report.md) — sample e2e test report after the heal phase
- [`skills-comparison.md`](./skills-comparison.md) — with-vs-without-skills comparison table

## App architecture

- **Frontend**: React 19 + Vite + Tailwind + shadcn/ui — invoice queue, multi-step form, line-item table, OTP approval dialog
- **Backend**: Azure Static Web Apps + Azure Functions (`api/`) with Cosmos DB for storage
- **Local mode**: the frontend works without the backend — API calls are mocked via an in-memory store, so `npm run dev` alone is enough to demo the skill

## See also

- [The `verify` skill on its own](../skill/) — if you just want the skill files to copy into your own repo
- [Playwright Workspaces docs](https://aka.ms/pww/docs)
