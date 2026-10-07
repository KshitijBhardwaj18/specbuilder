---
name: inter-build
description: Step 4 of 4 in the inter pipeline (inter-scope → inter-prd → inter-spec → inter-build). Turns the current milestone's approved plan into ordered end-to-end slices in inter/m<N>-<slug>/checklist.md, then builds them one at a time (verify, commit, tick), runs a hands-on final review, and finishes with a secrets check, a push and a PR. Each PR is then reviewed by a separate agent that has the full plan context, the agreed fixes are pushed, and inter/product.md is updated. Resumes at the first unchecked box. Use after inter-spec, whenever a build session was interrupted, or when the user types /inter-build.
---

# inter-build — Build It

You are a build strategist for ten minutes, then an executor. The thinking already happened in scope, PRD and spec. Here you turn it into ordered slices, get the order approved, and work through them, verifying and committing each one. One invocation can carry the whole build, and an interrupted session resumes where it stopped.

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

If the user brings a new idea, add it to **Ideas** in `product.md` and carry on. Planning a new milestone is the job of `inter-scope`.

If the next step is another skill, name it and stop, unless the user explicitly wants to redo this step. In that case, edit the existing doc rather than starting over.

Inside this skill:

- No `checklist.md` → go to **Git**, then **Plan the Slices**.
- Checklist `draft` → go to **Approve the Order**.
- Checklist `approved` with unchecked slices → go to **The Loop**, starting at the first unchecked slice.
- All slices checked → go to **Final Review**, then **Finish**, doing only the unchecked parts.

## Git

- Not a git repo → offer `git init`, and do it only on a yes.
- On the default branch → create `feat/<slug>` before the first commit.
- Make sure `.gitignore` covers `.env`, `.env.*` (keeping `.env.example`) and other local credential files. Ignoring a file doesn't untrack it, so check what's already tracked.
- Before every commit, read the staged diff for secrets and unrelated changes. Stage only the files you intend to commit, never with a blind `git add -A`.

## Before You Start

Read `spec.md` (note which headings actually exist, since every ref you write must point at a real one), the PRD's **Core Journey** and **Acceptance Criteria**, the milestone scope's **In**, **Out** and **Done Looks Like**, and the **Kernel** and **Cut** sections of `product.md`. Load further sections per slice, as the slice's refs point at them.

## Plan the Slices

A **slice** is a thin end-to-end increment that makes one real behaviour work. It crosses whatever layers it needs, and it can be run and checked the moment it's done. It is never a layer on its own.

- **Wrong (layers):** set up project → data model → API → UI → wire it up. Nothing works until the last step.
- **Right (slices):** you can add an entry and see it listed (scaffold included) → entries survive a restart → you can filter them. Each one runs.

**Exception:** a single technical layer can be its own slice only when it proves a critical risk on its own and leaves runnable evidence, for example a script confirming an unfamiliar API returns what the spec assumes.

Order the slices by these rules:

- **The milestone's core comes early.** In M1 that is the kernel; polished scaffolding around a missing kernel is a failed build.
- **Risk comes first:** the spec's **Open Issues** and highest **Risks and Caveats** get proven in the earliest slices they can be.
- **Bootstrapping lives inside slice 1** (M1 only). Never make it a step of its own.
- **The project is never left broken.** Every slice leaves something working.

Use as few slices as do the job. Prefer a few substantial slices to a ceremonial list.

Write `inter/m<N>-<slug>/checklist.md` with `status: draft`. **The field labels are a parsing contract, so reproduce them exactly.**

```markdown
---
doc: checklist
status: draft
---
# <Name> — Build Checklist

## Slices

- [ ] **1. <What becomes possible, in user terms>**
  Delivers: <the behaviour that now works>
  Why now: <why it sits in this position>
  Refs: `prd.md > <heading>`, `spec.md > <heading>`
  Build: <what to implement>
  Verify: <the exact command or observation you run and check yourself>
  Try it: <what the user opens, does and should see>
  Commit: `<message>`

## Final Review

- [ ] Hands-on review done, agreed fixes made, user confirms it's ready

## Finish

- [ ] Full checks pass (tests, lint, build per `spec.md > Run and Verify`)
- [ ] Secrets and history check clean
- [ ] README and `.env.example` reflect how to run it
- [ ] Branch pushed and PR opened: <link>
- [ ] Code review done: findings triaged, fixes pushed
- [ ] `product.md` updated: milestone logged under Shipped, learnings moved into Ideas

## Revisions
```

## Approve the Order

Show the slices in order: what works after each one, and why it sits where it does. Make the sequencing logic visible, not just the list. Ask once: "Does this order look right?" If the feedback reaches into product or architecture, fix `prd.md` or `spec.md` in the same pass so the docs never contradict each other. A new feature in the feedback is a scope change: name its cost, or park it in **Ideas** in `product.md`. On a clear yes, set `status: approved`. **Don't write code before that.**

## The Loop

For each unchecked slice, in order:

1. **Build** what `Build:` says, guided by `Refs:`. Stay within the spec's **Decisions**, follow its **Testing Strategy**, and respect its **Rollout** order for migrations and flags. Follow the existing codebase's conventions and `spec.md > Look and Feel`, not framework defaults. Write it the way you'd want to maintain it.
2. **Verify** by running exactly what `Verify:` says, and read the result. "Should work" is not verification. Check the PRD acceptance criteria this slice touches.
3. **Repair** before moving on. Never carry a known failure forward. If you can't fix it, go to **Recovery**.
4. **Commit** with the `Commit:` message. Do this every slice, without asking.
5. **Tick** the box (`- [ ]` → `- [x]`) right away. This file is the progress state; stale state is worse than none.
6. **Continue** without asking for permission.

**Pause for the user to try it** (using the slice's `Try it:`) after slice 1, at any slice that is the first chance to see the kernel, and whenever their feedback could change the remaining slices. Ask what they noticed and what they'd change, then fix, re-verify and commit before you continue. Otherwise keep moving with brief narration: what you're building, and when it works.

## When Reality Disagrees with the Plan

It will. A library won't behave, a data shape won't fit, or a step needs a piece nobody planned.

1. Check the impact across scope, PRD, spec and the later slices.
2. Update only the affected sections and slices.
3. Add a bullet under `## Revisions`: what changed, and what the build discovered.
4. Continue.

An internal correction (a moved file, a different helper) needs no conversation. A change to what the user gets, to the chosen stack, or to the **Kernel** needs their agreement first.

## Recovery

The last commit is the recovery point. Never discard the user's work, and never hard-reset, force-push, clean untracked files or check out over uncommitted changes without saying exactly what would be lost and getting a yes. Prefer fixing forward, then reverting one specific change, then setting aside this slice's changes. When stuck, stop and say what you tried and what failed, propose the smallest safe step back, and treat the result as a plan revision.

## Final Review

Once every slice is checked:

1. Run the full checks from `spec.md > Run and Verify`, and walk every PRD acceptance criterion.
2. Have the user run the app and explore it freely: the core journey, odd inputs, rough edges.
3. Interview before you change anything: "What happened, and what did you expect?" Don't jump from a vague complaint to your own redesign.
4. Add each agreed fix as an unchecked item under **Final Review**, then build it, verify it, commit it and tick it.
5. Tick the review box only when the user confirms it's ready. "Nothing to change" is a valid outcome.

## Finish

1. **Checks:** run the full test, lint and build suite one last time.
2. **Secrets:** scan the branch's diff and history for keys, tokens, `.env` contents and private data. If anything is found, stop and agree on a fix; exposed credentials need rotating. Never rewrite history silently.
3. **Docs:** make sure the README explains how to run it and `.env.example` lists every variable, with placeholders only.
4. **Ship:** with a yes from the user, push the branch and open a PR that summarises what was built, links `inter/m<N>-<slug>/`, and notes any revisions. If the diff is too big to review in one go, propose splitting it into stacked PRs along slice lines.
5. **Review:** run **Code Review** below for each PR before going on.
6. **Update `product.md`:** add `M<N> <name>: <one line> (<PR link>)` under **Shipped**. Put anything the build surfaced (deferred fixes, revisions that point at future work, ideas the user raised) under **Ideas**. Don't plan the next milestone here; that's for `inter-scope`.

Tick each **Finish** box as it lands, then set the checklist to `status: done`.

## Code Review

Review every PR, and every PR in a stack separately, with a **fresh agent**. The builder is too close to its own code, so the review has to come from a context that didn't write it.

**1. Spawn the reviewer.** Start a separate subagent with no build history. Give it file paths, not summaries, so it reads everything first-hand:

- the PR number, plus its base and head branch;
- `inter/product.md`;
- the milestone's `scope.md`, `prd.md`, `spec.md` and `checklist.md`, including the **Revisions**;
- for a stacked PR, the slices that PR covers.

**2. Brief the reviewer.** Tell it to read all of those docs before the diff, then review the PR:

- If the harness has a `/code-review` skill, run it on the PR at high effort, without posting comments. Then layer the plan checks below on top of its findings.
- Otherwise, review the diff by hand, covering both correctness and the plan checks.

The plan checks:

- **Spec conformance:** every **Decision** is implemented as agreed, every contract in **Data Model** and **Interfaces** matches, and nothing outside the spec slipped in.
- **Acceptance criteria:** each PRD criterion is actually met by the code, not just claimed in the checklist.
- **Risks:** each item under **Risks and Caveats** and **Rollout** has its mitigation in place, and migrations are safe and in the right order.
- **Security:** authentication and authorisation on every new path, input validation at trust boundaries, and no secrets.
- **Tests:** the critical paths named in **Testing Strategy** are actually covered.
- **Conventions:** the new code reads like the surrounding codebase, and no existing helper was re-implemented.

The reviewer returns its findings ranked from most to least severe. Each finding has a `file:line`, what's wrong, the concrete failure it causes, and a suggested fix. It changes no files and posts nothing.

**3. Triage with the user.** Show the findings. For each one, the user decides between three outcomes:

- **Fix:** build it, verify it, commit it, and push it to the PR branch.
- **Defer:** add it to **Ideas** in `product.md` with a link to the PR.
- **Dismiss:** record the reason it doesn't apply.

Record every finding and its outcome as a line under **Final Review** in the checklist.

**4. Re-review if needed.** If any fix touched a one-way-door decision, or changed more than a few lines, spawn a fresh reviewer for the fix commits only. Otherwise one round is enough.

Only post the findings to the PR as comments if the user asks for that.

## Hand Off

"M<N> is shipped: <PR link>. `inter/m<N>-<slug>/` holds the full record, and `product.md` is updated. When you're ready, `inter-scope` will help pick the next milestone."
