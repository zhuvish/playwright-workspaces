# App Verification Skills — Invoice Processing demo

A complete, runnable demo app that ships with the **`verify`** Copilot CLI skill pre-wired. Clone the repo, run the app, and ask Copilot CLI to verify it — the skill drives a full QA pass on Playwright Workspaces cloud browsers.

## What's in this sample

| Part | Where | What it is |
|------|-------|------------|
| **Demo app** | [`src/`](./src/), [`api/`](./api/), [`tests/`](./tests/) | A small React + Vite invoice-processing app (multi-step form, line-item table, OTP approval, etc.) with existing Playwright tests in [`tests/`](./tests/) |
| **The skill** | [`.github/skills/verify/`](./.github/skills/verify/) | The `verify` skill (SKILL.md + 14 reference docs) — Copilot CLI auto-detects any folder under `.github/skills/` |
| **App docs** | [`PRD.md`](./PRD.md), [`docs/`](./docs/) | Product spec and architecture notes for the demo app |
| **Background reading** | [`testing-skills.md`](./testing-skills.md), [`skills-comparison.md`](./skills-comparison.md), [`report.md`](./report.md) | Hand-written verification report, with-vs-without-skills comparison table, and a sample e2e test report so you can see the kind of output the skill produces |

## What the `verify` skill does

Three-phase end-to-end testing pipeline driven by Copilot CLI:

1. **App verification** — drives a real browser via `playwright-cli`, exercises the core flows, captures snapshots and console errors
2. **Test authoring** — generates Playwright tests for working flows that aren't already covered
3. **Test healing** — re-runs the existing suite, classifies failures (`stale_test` vs real `regression`), fixes the stale ones, and surfaces genuine regressions with evidence

All test execution happens on **Playwright Workspaces** cloud browsers — no local browser install required, parallelizable, with video + trace artifacts uploaded for inspection.

## Try it on this demo

### 1. Clone and install

```bash
git clone https://github.com/Azure/playwright-workspaces.git
cd playwright-workspaces/samples/app-verification-skills
npm install
```

### 2. Start the app

```bash
npm run dev   # vite serves at http://localhost:5001
```

### 3. Prerequisites for the skill

- **Azure subscription** with permissions to create Playwright Workspaces — [quickstart](https://learn.microsoft.com/en-us/azure/app-testing/playwright-workspaces/quickstart-run-end-to-end-tests?tabs=playwrightcli&pivots=playwright-test-runner)
- **Playwright Workspace resource ID** — get one from the Azure portal
- **Azure CLI** signed in (`az login`)
- **Copilot CLI** installed and signed in

On first invocation the skill bootstraps its config files so subsequent runs are zero-setup.

### 4. Run Copilot in this folder and invoke the skill

```bash
copilot
# then: /verify
```

Copilot picks up `.github/skills/verify/` automatically — no plugin install, no manual registration.

## Use it on your own app

The skill is just a folder. Drop it into your repo:

```bash
# from the root of your own project
cp -r path/to/playwright-workspaces/samples/app-verification-skills/.github/skills .github/
```

Resulting layout in your repo:

```
your-repo/
└── .github/
    └── skills/
        └── verify/
            ├── SKILL.md
            └── references/
                ├── phase-1-app-verification.md
                ├── phase-2-test-authoring.md
                ├── phase-3-test-healing.md
                └── ...
```

Then run `copilot` in your repo and invoke `/verify`.

## What you get out

A single consolidated report covering:

- ✅ Which app flows are working (with screenshots/traces)
- 📝 Which new tests were authored, with the file paths
- 🩹 Which failing tests were healed (real fixes, not `test.skip`)
- 🚨 Which failures are genuine regressions you need to look at, with evidence

See [`report.md`](./report.md) and [`testing-skills.md`](./testing-skills.md) in this folder for examples of what those reports look like on the demo app.

## Use cases

Triggered by asks like *"verify my app", "make sure my app works and has good test coverage", "do a full QA pass before deploy", "add tests where I'm missing coverage", or "QA my deployment"*.

Not for unit tests, API tests, native mobile/desktop apps, load/perf testing, or full accessibility audits — use the right tool for the job.

## More resources

- [Playwright Workspaces docs](https://aka.ms/pww/docs)
- [With vs without skills — comparison table](./skills-comparison.md)
