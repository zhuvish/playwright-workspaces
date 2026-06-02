# App Verification Skills

This sample showcases the **`verify`** Copilot CLI skill — an end-to-end app verification pipeline that uses Playwright Workspaces (PWW) as the cloud browser backend.

The sample is split into two folders so you can use whichever piece you need:

| Folder | What it contains | When to use it |
|--------|------------------|----------------|
| **[`skill/`](./skill/)** | The `verify` skill (SKILL.md + 14 reference docs) on its own | You already have your own app — lift `skill/verify/` into your repo's `.github/skills/` and you're done |
| **[`sample-app/`](./sample-app/)** | A complete runnable invoice-processing demo app with the skill **pre-wired** at `.github/skills/verify/` | You want to see the skill in action on a real app before adopting it |

## What the `verify` skill does

Three-phase end-to-end testing pipeline driven by Copilot CLI:

1. **App verification** — drives a real browser via `playwright-cli`, exercises the core flows, captures snapshots and console errors
2. **Test authoring** — generates Playwright tests for working flows that aren't already covered
3. **Test healing** — re-runs the existing suite, classifies failures (`stale_test` vs real `regression`), fixes the stale ones, surfaces genuine regressions with evidence

All test execution happens on **Playwright Workspaces** cloud browsers — no local browser install required, parallelizable, with video and trace artifacts uploaded for inspection.

## Quickstart paths

**Try it on the bundled demo app** (the fastest way to see what the skill does):

```bash
cd sample-app
npm install
npm run dev        # vite serves at http://localhost:5001 in another terminal
copilot
# then: /verify
```

See [`sample-app/README.md`](./sample-app/README.md) for details.

**Use it on your own app** (lift the skill folder into your repo):

```bash
# from the root of your own project
cp -r path/to/playwright-workspaces/samples/app-verification-skills/skill/verify .github/skills/
```

See [`skill/README.md`](./skill/README.md) for details.

## More resources

- [Playwright Workspaces docs](https://aka.ms/pww/docs)
