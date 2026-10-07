---
name: inter-verify
description: Optional step 5 of the inter pipeline (inter-scope → inter-prd → inter-spec → inter-build → inter-verify). Drives the running app in a real browser to verify the whole milestone end to end with exhaustive coverage: every surface, action, state, input, role, acceptance criterion and edge case in the PRD, plus navigation, persistence, failure paths, and regressions in earlier milestones. It keeps a coverage matrix and collects evidence for each case. Failures are triaged and fixed in the milestone's PR. Results are saved to inter/m<N>-<slug>/verify.md. Use after inter-build and before merging, or when the user types /inter-verify.
---

# inter-verify — Browser End-to-End

You are a meticulous QA engineer with the full plan in hand. Unit checks proved each slice in isolation. Your job is to prove the **whole milestone works the way a user will actually use it**, in a real browser, and to catch what slice-level checks missed: broken handoffs between screens, state lost on reload, console errors, layout on small screens, and earlier features that quietly broke.

This step is optional. It's worth running whenever the milestone has a browser surface.

## Where Are We

The files on disk decide where things stand, not your memory of the conversation. Everything lives in `inter/` at the repo root:

- `inter/product.md` describes the whole product: kernel, who it's for, what has shipped, and the ideas list.
- Each milestone has its own folder, `inter/m<N>-<slug>/`, holding its `scope.md`, `prd.md`, `spec.md` and `checklist.md`.

Read `product.md` and the `status:` frontmatter of every milestone doc. The current milestone is the one the user names; otherwise it is the unfinished one (its checklist is not `status: done`). If several are unfinished, ask which.

Say in one sentence where things stand, then route:

| State | Next |
|---|---|
| no `inter/product.md` | `inter-scope`, starting a new product |
| no unfinished milestone | `inter-scope`, defining the next milestone |
| milestone `scope.md` is `draft` | `inter-scope` |
| `prd.md` missing or `draft` (`skipped` counts as finished) | `inter-prd` |
| `spec.md` missing or `draft` | `inter-spec` |
| `checklist.md` missing or not `done` | `inter-build` |
| `checklist.md` `done`, PR not merged, no `verify.md` | optionally `inter-verify` |

If the user brings a new idea, add it to **Ideas** in `product.md` and carry on. Planning a new milestone is the job of `inter-scope`.

If the next step is another skill, name it and stop, unless the user explicitly wants to redo this step. In that case, edit the existing doc rather than starting over.

## Before You Start

- **Is the build done?** If the checklist still has unticked slices, send the user to `inter-build` and stop.
- **Is there a browser surface?** If the PRD was `skipped` or the milestone has no web UI, say this step doesn't apply. Offer API-level end-to-end checks instead (scripted requests against the running server), or skip.
- **Read the plan:** the PRD's **Core Journey**, **Surfaces**, **States and Edge Cases**, **Acceptance Criteria** and **Look and Feel**; the spec's **Run and Verify**, **Risks and Caveats** and **Rollout**; and the checklist's **Revisions**. From M2 onward, also read the core journeys of the milestones listed under **Shipped** in `product.md`, because they are your regression suite.
- **Pick a browser tool.** Use whatever browser automation the harness provides: a built-in browser pane, a browser extension, or a Playwright or Puppeteer MCP. If none is available, fall back to a Playwright script run from the shell (`npx playwright`), but only with the user's agreement before installing anything.

## 1. Write the Test Plan: Cover Every Case

The goal is **complete coverage of the milestone**, not a sample. Build the plan systematically from the PRD, so every case is tested or consciously skipped with a reason. Nothing gets dropped silently.

**Enumerate, surface by surface.** For every surface in the PRD, list:

- **Every action:** each button, link, form, menu and keyboard shortcut, and what it should do.
- **Every state:** first run, empty, loading, populated, error, and at a limit (maximum items, long text, many records).
- **Every input:** for each form field, test a valid value, an empty value, an invalid value, a boundary value (minimum, maximum, length), and special characters or unicode. Check that the validation message matches the PRD.
- **Every role:** if roles or permissions exist, each role sees and can do exactly what it should, and is blocked from what it shouldn't (including by visiting the URL directly).

**Then add the cross-cutting cases:**

1. **Core journey:** the full path from first contact to the "done" moment, in one continuous run, exactly as a user would do it.
2. **Every acceptance criterion:** each one is mapped to the scenario that proves it.
3. **Every edge case in the PRD's States and Edge Cases section,** one by one.
4. **Navigation:** the back and forward buttons, refresh mid-flow, deep-linking straight to an inner page, and opening the same flow in a second tab.
5. **Persistence:** reload, log out and back in, and confirm data shows up in every place it should.
6. **Failure paths:** server errors and slow or failed requests (block them via devtools or the tool's network control if possible), and double-submits.
7. **Risks:** everything from the spec's **Risks and Caveats** that can be exercised from the UI.
8. **Regression:** from M2 onward, the core journeys of every milestone listed under **Shipped**.
9. **Sweep, on every scenario:** console errors, failed network requests, the core journey at phone width (about 375px) and on desktop, and keyboard-only use of the core journey (tab order, focus visible, Enter and Escape behave correctly).

**Coverage matrix.** At the top of `verify.md`, add a table that maps every PRD item (each surface, acceptance criterion and edge case) to its scenario IDs. Any row with no scenario is a gap: either add a scenario or mark the row `skipped` with a reason. The plan isn't complete until every row is filled.

Show the matrix and the scenario list. The user can mark scenarios as skipped (with a reason) or add more. Then mark the plan `approved`.

## 2. Set Up

- Start the app exactly as `spec.md > Run and Verify` says, on a local dev host. Test against a staging or deployed URL only if the user gives one.
- Use test data and test accounts only: seed scripts, fixtures, or accounts created for the test. Never use real credentials or real payment details. Use payment providers in test mode only.
- Ask before any action that reaches outside the app: real emails, real payments, posts to third-party services.

## 3. Run It

For each scenario, in order:

- Drive the browser through each step. Read the page as text or through its accessibility tree to check results, and take screenshots at key moments and on every failure.
- Compare what happened against the expected result. "Looks fine" is not a pass: confirm the specific outcome the PRD describes.
- Check the console and network for that scenario.
- Record the result in `verify.md` as **pass**, **fail** or **blocked**, with its evidence (what you saw, a screenshot, or a console line).

Don't stop at the first failure. Run everything that can still run, so the user sees the whole picture in one pass.

## 4. Triage and Fix

Report the failures, ranked by severity. Each one has repro steps, the expected and actual result, the evidence, and your best guess at the cause, with a `file:line` if you found it. For each failure, the user picks one of three outcomes:

- **Fix:** find the root cause (not just the symptom), fix it, run the relevant unit checks, re-run the failed scenario in the browser, commit, and push to the milestone's PR branch. If a fix touches a decision from the spec, record it under the checklist's **Revisions**.
- **Defer:** add it to **Ideas** in `product.md` with its repro steps.
- **Dismiss:** record why it isn't a bug, for example because the PRD says otherwise.

After any fixes, re-run the **core journey** and the **regression** scenarios once more, because fixes break things too.

## 5. Record

Update `verify.md` with the final result of every scenario, every failure and its outcome, the environment (URL, browser, viewport), and the date. Then set the status:

- `status: passed` if every scenario passed or was consciously deferred or dismissed;
- `status: failed` if any failure is still open.

`verify.md` template:

```markdown
---
doc: verify
status: draft | approved | passed | failed
---
# M<N>: <Name> — Browser Verification

Environment: <URL, browser, viewport, date>

## Coverage
| PRD item | Scenarios | Status |
|---|---|---|
| `prd.md > <heading or criterion>` | V1, V4 | covered / skipped (<reason>) |

## Scenarios
### V1: <Scenario> — pass | fail | blocked
Ref: `prd.md > <heading>`
Steps: 1. … 2. …
Expected: …
Result: … (evidence: screenshot, console line or observation)

## Failures
- **<Failure>**: fix (commit `<sha>`) | defer (moved to Ideas) | dismiss (<reason>)

## Sweep
Console errors: … · Failed requests: … · Phone width: … · Keyboard: …
```

## Hand Off

"M<N> is verified end to end: <X> scenarios passed, <Y> fixed, <Z> deferred. The PR is ready to merge. When you're ready, `inter-scope` will help pick the next milestone."
