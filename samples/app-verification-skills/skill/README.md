# The `verify` Skill

This folder contains the **`verify`** Copilot CLI skill on its own — `SKILL.md` plus 14 reference docs covering the three pipeline phases, cloud setup, credential handling, recovery paths, and report formats.

For a runnable demo of the skill against a real app, see the sibling [`sample-app/`](../sample-app/) folder.

## What the skill does

Three-phase end-to-end testing pipeline driven by Copilot CLI:

1. **App verification** — drives a real browser via `playwright-cli`, exercises the core flows, captures snapshots and console errors
2. **Test authoring** — generates Playwright tests for working flows that aren't already covered
3. **Test healing** — re-runs the existing suite, classifies failures (`stale_test` vs real `regression`), fixes the stale ones, surfaces genuine regressions with evidence

All test execution happens on Playwright Workspaces cloud browsers — no local browser install required, parallelizable, with video + trace artifacts uploaded for inspection.

## Install in your own repo

Copilot CLI auto-detects any skill folder placed under `.github/skills/`. Copy `verify/` into your repo:

```bash
# from the root of your own project
cp -r path/to/playwright-workspaces/samples/app-verification-skills/skill/verify .github/skills/
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

Then run Copilot CLI in your repo and invoke the skill:

```bash
copilot
# then: /verify
```

No plugin install, no manual registration — Copilot CLI picks it up on the next session.

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

## Skill folder contents

| File | Purpose |
|------|---------|
| `verify/SKILL.md` | Entrypoint — Copilot CLI reads this first |
| `verify/references/installation.md` | Tool detection + install prerequisites |
| `verify/references/cloud-setup.md` | Playwright Workspace + Azure auth setup |
| `verify/references/credential-handling.md` | App-login conventions (env vars, TOTP, never hardcode) |
| `verify/references/sensitive-data.md` | What to redact from reports |
| `verify/references/repo-discovery.md` | How to find the right entry point + existing tests |
| `verify/references/app-launch.md` | Local dev-server start patterns |
| `verify/references/project-config-conventions.md` | Where to write per-project config |
| `verify/references/phase-1-app-verification.md` | Phase 1 — drive the app, capture behaviour |
| `verify/references/phase-2-test-authoring.md` | Phase 2 — generate Playwright tests for uncovered flows |
| `verify/references/phase-3-test-healing.md` | Phase 3 — classify and fix failing tests |
| `verify/references/final-sanity-run.md` | Re-run the full suite after fixes |
| `verify/references/recovery.md` | Failure-recovery paths (browser crash, auth block, network) |
| `verify/references/verification-report.md` | Hand-written verification-report template |
| `verify/references/consolidated-report.md` | Final consolidated-report template |

## Use cases

Triggered by asks like *"verify my app", "make sure my app works and has good test coverage", "do a full QA pass before deploy", "add tests where I'm missing coverage", or "QA my deployment"*.

Not for unit tests, API tests, native mobile/desktop apps, load/perf testing, or full accessibility audits — use the right tool for the job.

## More resources

- [Playwright Workspaces docs](https://aka.ms/pww/docs)
- [Runnable demo with this skill pre-wired](../sample-app/)
