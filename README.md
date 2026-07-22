# BC Page Scripting Tests

CI pipeline that replays Business Central page-scripting recordings against a live BC web client using Microsoft's
stand-alone player, [`@microsoft/bc-replay`](https://www.npmjs.com/package/@microsoft/bc-replay) (built on
Playwright/Chromium). It runs on pull request, push to `main`, and manual dispatch, and uploads the HTML report,
video, and traces as a downloadable artifact.

## ⚠️ Before you run this: two things that make or break it

1. **Reachability** — the GitHub-hosted runner must be able to reach your BC URL over the network. BC SaaS/online is
   reachable from `ubuntu-latest` runners out of the box. On-prem/private BC (VPN-only, firewalled) needs a
   **self-hosted runner** — change `runs-on:` in the workflow accordingly.
2. **No MFA** — the page scripting player signs in with a plain username + password, it does not handle interactive
   MFA prompts. For SaaS (`AAD` auth) you need a **dedicated automation user account** that is:
   - excluded from Conditional Access MFA policy,
   - assigned the **`PAGESCRIPTING - PLAY`** permission set,
   - pointed at a **sandbox environment — never production**.

Resolve both of these before enabling scheduled or push-triggered runs.

## Setup

1. **Add repo secrets** (Settings > Secrets and variables > Actions > Secrets):
   - `BC_USERNAME`
   - `BC_PASSWORD`
2. **Add repo variables** (same page, Variables tab):
   - `BC_START_ADDRESS` — the BC web client URL to test
   - `BC_AUTHENTICATION` — `Windows`, `AAD`, or `UserPassword` (defaults to `UserPassword` if unset)
3. **Add recordings** under [`recordings/`](recordings/README.md) — at least one `.yml` file is required or the run
   fails fast with a clear error.
4. Push to `main` (or open a PR touching `recordings/**`), or trigger **Actions > BC Page Scripting Tests > Run
   workflow** for a manual run (optionally overriding the start address for that run only).

## Viewing results

Every run uploads a `bc-replay-results` artifact (retained 14 days) containing the HTML report, videos, and traces.
Download it, unzip it, then open the report with:

```bash
npx playwright show-report ./results/playwright-report
```

## Running locally

```powershell
npm install
npx playwright install --with-deps chromium
New-Item -ItemType Directory -Force -Path ./results | Out-Null
& "./node_modules/@microsoft/bc-replay/Replay.ps1" "./recordings/*.yml" -StartAddress "https://<your-bc-url>" -Authentication UserPassword -UserNameKey BC_USERNAME -PasswordKey BC_PASSWORD -Headed -ResultDir ./results
```

`-Headed` opens a visible browser window for local debugging; omit it for a headless run. `BC_USERNAME` /
`BC_PASSWORD` must be set as environment variables locally (the `-UserNameKey` / `-PasswordKey` arguments name the
env vars to read, not the literal credentials).

## `npx replay` parameters used by this pipeline

| Parameter | Description |
|---|---|
| `<glob>` (positional) | Glob of `.yml` recording files to replay, e.g. `./recordings/*.yml` |
| `-StartAddress <url>` | BC web client URL to test against |
| `-Authentication Windows\|AAD\|UserPassword` | Authentication mode |
| `-UserNameKey <envName>` | Name of the environment variable holding the username |
| `-PasswordKey <envName>` | Name of the environment variable holding the password |
| `-Headed` | Run with a visible browser window (local debugging only, not used in CI) |
| `-ResultDir <dir>` | Output directory for the HTML report, videos, and traces (report lands at `<dir>/playwright-report`) |

## Scope and limitations

The Business Central page scripting tool is a production-ready **preview** feature. It only captures **AL-driven UI**
— it cannot record or replay interactions inside control add-ins or embedded Power BI visuals. For coverage of
business logic that isn't exercised through the UI (or that needs to run faster/more granularly than a UI replay),
pair this with **AL automated/unit tests** run via **BcContainerHelper** or **AL-Go for GitHub** — that is a separate,
complementary testing concern from this UI-replay pipeline.
