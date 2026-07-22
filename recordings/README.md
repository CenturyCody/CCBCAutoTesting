# Recordings

This folder holds the page scripting recordings (`.yml`) that CI replays with `@microsoft/bc-replay`.

## Capturing a recording

1. Sign in to the Business Central web client with an account that has the **`PAGESCRIPTING - REC`** permission set.
2. Open the Settings (cog) menu > **Page Scripting** > **Start new**.
3. Perform the steps you want to automate, starting from a known page (ideally the Role Center — see below).
4. Open Page Scripting again and choose **Stop**, then **Save** to download the `.yml` file.
5. Drop the file into this folder (or a subfolder) and give it a descriptive, numbered name.

## Assert, don't just click

A recording that only clicks through UI proves the app didn't crash — it doesn't prove the data is correct. Add explicit
validation steps (field value checks, page/message assertions) while recording, so a regression that produces wrong data
still fails the test even if every click succeeds.

## Keep recordings small and independent

Prefer many short, focused recordings over one long one:

- `01-create-customer.yml`
- `02-create-sales-order.yml`
- `03-post-sales-order.yml`

Small recordings are easier to debug when they fail, and a broken step only takes out one test instead of an entire
end-to-end chain. A recording can reference/include another recording to compose a suite (e.g. a "post sales order" test
that first includes "create customer" and "create sales order" as setup) — check the current `bc-replay` docs for the
include syntax.

## Avoiding flaky runs

- Always start from the Role Center, not a mid-navigation state — the player needs a consistent, known starting page.
- Create fresh data in each recording (new customer, new document, etc.) rather than depending on records that may or
  may not exist from a previous run.
- Parameterize values that change between runs (numbers, dates, generated names) instead of hardcoding values that
  will collide or go stale over time.
