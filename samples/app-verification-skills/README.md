# App Verification Skills

This sample showcases the **`verify`** Copilot CLI skill — an end-to-end app verification pipeline that uses Playwright Workspaces (PWW) as the cloud browser backend.

Everything lives under [`sample-app/`](./sample-app/): a complete runnable invoice-processing demo app with the skill **pre-wired** at [`sample-app/.github/skills/verify/`](./sample-app/.github/skills/verify/). Copilot CLI auto-detects it the moment you `cd` into the folder — no plugin install, no manual registration.

## What the `verify` skill does

Three-phase end-to-end testing pipeline driven by Copilot CLI:

1. **App verification** — drives a real browser via `playwright-cli`, exercises the core flows, captures snapshots and console errors
2. **Test authoring** — generates Playwright tests for working flows that aren't already covered
3. **Test healing** — re-runs the existing suite, classifies failures (`stale_test` vs real `regression`), fixes the stale ones, surfaces genuine regressions with evidence

All test execution happens on **Playwright Workspaces** cloud browsers — no local browser install required, parallelizable, with video and trace artifacts uploaded for inspection.

## Two ways to use this

**1. Try it on the bundled demo app** (the fastest way to see what the skill does):

```bash
cd sample-app
npm install
npm run dev          # vite serves at http://localhost:5001 in another terminal
copilot
# then: /verify
```

Full instructions and demo details are in [`sample-app/README.md`](./sample-app/README.md).

**2. Lift the skill into your own repo**:

```bash
# from the root of your own project
cp -r path/to/playwright-workspaces/samples/app-verification-skills/sample-app/.github/skills/verify .github/skills/
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

Then run `copilot` in your repo and invoke `/verify`. The skill picks itself up — no plugin install, no manual registration.

## Prerequisites

- **Azure subscription** with permissions to create Playwright Workspaces — [quickstart](https://learn.microsoft.com/en-us/azure/app-testing/playwright-workspaces/quickstart-run-end-to-end-tests?tabs=playwrightcli&pivots=playwright-test-runner)
- **Playwright Workspace resource ID** — get one from the Azure portal
- **Azure CLI** signed in (`az login`)
- **Copilot CLI** installed and signed in

On first invocation the skill bootstraps its config files so subsequent runs are zero-setup.

## What you get out

A single consolidated report covering:

- ✅ Which app flows are working (with screenshots/traces)
- 📝 Which new tests were authored, with the file paths
- 🩹 Which failing tests were healed (real fixes, not `test.skip`)
- 🚨 Which failures are genuine regressions you need to look at, with evidence

See [`sample-app/testing-skills.md`](./sample-app/testing-skills.md) and [`sample-app/report.md`](./sample-app/report.md) for examples of what those reports look like on the demo app.

## Use cases

Triggered by asks like *"verify my app", "make sure my app works and has good test coverage", "do a full QA pass before deploy", "add tests where I'm missing coverage", or "QA my deployment"*.

Not for unit tests, API tests, native mobile/desktop apps, load/perf testing, or full accessibility audits — use the right tool for the job.

## More resources

- [Playwright Workspaces docs](https://aka.ms/pww/docs)
- [With vs without skills — comparison table](./sample-app/skills-comparison.md)

