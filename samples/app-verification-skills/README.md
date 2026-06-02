# App Verification Skills — Verify Skill

This sample ships the **`verify`** Copilot CLI skill — an end-to-end app verification pipeline that uses Playwright Workspaces (PWW) as the cloud browser backend.

The skill itself lives in [`skills/verify/`](./skills/) next to this README. Drop it into your repo's `.github/skills/` folder and Copilot CLI picks it up automatically — no plugin install, no separate registration.

## What the verify skill does

`verify` is a three-phase end-to-end testing pipeline driven by Copilot CLI:

1. **App verification** — exercises the running app to confirm the core flows still work
2. **Test authoring** — generates Playwright tests for any flows that are working but not covered
3. **Test healing** — re-runs the existing suite, classifies failures (stale test vs real regression), fixes the stale ones, surfaces the regressions

All test execution happens on Playwright Workspaces cloud browsers — no local browser install required, parallelizable, with video and trace artifacts uploaded for later inspection.

## Install in your repo

Copy the `skills/` folder into your project's `.github/`:

```bash
# from the root of your own project
cp -r path/to/playwright-workspaces/samples/app-verification-skills/skills .github/
```

Resulting layout:

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

Then run Copilot CLI in your repo and invoke the skill:

```bash
copilot
# then: /verify
```

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

## Use cases

Triggered by asks like *"verify my app", "make sure my app works and has good test coverage", "do a full QA pass before deploy", "add tests where I'm missing coverage", or "QA my deployment"*.

Not for unit tests, API tests, native mobile/desktop apps, load/perf testing, or full accessibility audits — for those, use the right tool for the job.

## More resources

- [Playwright Workspaces docs](https://aka.ms/pww/docs)


