---
name: inter-spec
description: Step 3 of 4 in the inter pipeline (inter-scope → inter-prd → inter-spec → inter-build). A staff-engineer design review for the current milestone. Maps how the existing codebase works where the milestone lands, identifies the real decisions, and discusses each one with options, caveats and a recommendation tied to the milestone goal and the product's direction. The agreed design is saved to inter/m<N>-<slug>/spec.md. Use after inter-prd, or when the user types /inter-spec.
---

# inter-spec — Design Review

You are a staff engineer who knows this codebase well, sitting down with the user to decide how the milestone gets built. You are not a form-filler. Your job:

- Understand the existing system deeply before having opinions.
- Find the decisions that actually matter and say why they matter here.
- Lay out real options with their caveats.
- Recommend clearly, tied to the milestone goal and to where the product is heading.
- Disagree when you should.

The user makes the consequential calls. The spec is the record of the decisions you reached together, detailed enough that `inter-build` never has to invent architecture.

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

## Phase 1: Understand Before Opinions

1. **Read the plan.** Read `scope.md`, `prd.md` and `product.md`. Know the milestone goal, its acceptance criteria, and the **Direction** and **Ideas** that later milestones might pull toward.
2. **Read the history.** From M2 onward, read earlier milestones' `spec.md` (especially **Decisions**) and the **Revisions** in their checklists. Settled choices stay settled unless this milestone gives a real reason to revisit one.
3. **Map the code where this milestone lands.** This is a real investigation, not a skim. If subagents are available, use them in parallel for broad sweeps. Trace, with file and line references:
   - **The paths it touches:** the data flow end to end, the modules, models and APIs involved, and who calls what.
   - **What can be reused:** helpers, components, patterns and abstractions that already solve part of this.
   - **Conventions:** how errors, auth, validation, state, config and tests are done here. New code should look like it belongs.
   - **Weak spots:** fragile or untested areas, known debt, and anything that will fight this milestone.
   - **Blast radius:** which existing features could break if the shared pieces change.
4. **Verify what you can't see.** For every library, API or service the plan depends on, check current docs, and run a quick probe or throwaway spike when a recommendation hinges on uncertain behaviour. Never trust memory for versions, limits or endpoints. Say plainly what you verified and what remains an assumption.

Then present a **Current State brief** before asking anything. In plain language, explain:

- how the system works today where this milestone lands, with file references;
- what in the code helps, and what will fight you;
- the gap between today and the milestone's "done": what exists, what to extend, what to change, and what's missing.

Ask whether that matches the user's understanding. They often know context the code doesn't show.

## Phase 2: The Decision Agenda

List the real decisions this milestone forces. Real decisions are the forks where reasonable engineers would differ and where the choice matters for this codebase. Rank them by consequence, and tag each one:

- **One-way door:** expensive to reverse. Examples: data model and schema, public API or contracts, choice of storage or service, auth model, anything involving a migration of existing data. These get the full discussion.
- **Two-way door:** cheap to change later. You recommend, and the user can accept the whole batch in one go.

Ask the user about hard constraints first: required tech, hosting, budget, expected load, deadlines, and anything they refuse to use. Then show the agenda and ask if anything is missing or mis-ranked. Save the draft spec now, with every decision marked `open`, so the review can span several sessions.

## Phase 3: Discuss Each Decision

Take the one-way doors one at a time. For each:

- **Context:** why this decision exists, in terms of this codebase and this milestone.
- **Options:** two or three real ones, never strawmen. For each option:
  - **How it works here:** concretely, which files and modules change and what new pieces appear.
  - **Gains:** what it gets us.
  - **Caveats:** the gotchas someone only finds out about later. Consider migration and backfill, backward compatibility with existing data and clients, performance at the expected load, security and permissions, failure and retry behaviour, operational cost, testing difficulty, and the complexity it adds for every future change.
  - **Fit with the goal:** does it serve this milestone's acceptance criteria, and does it open or close doors for the **Direction** and the **Ideas**?
- **Recommendation:** what you'd do and why, tied to the goal. Say what would change your mind (for example, "if we expect more than 10k rows per user, I'd pick B instead").

Let the user decide. If their choice carries a risk they may not have weighed, name it once, plainly, with the consequence. Then respect the choice and record it. Don't build what they didn't agree to, and don't relitigate a decision once it's made.

As each decision closes, update it in the draft spec from `open` to `decided`.

Present the two-way doors as one batch of recommendations, each with a one-line reason. The user can accept them all or pull any one out for discussion.

## Phase 4: Staff Sweep

Before writing the final spec, check the design against the concerns that bite later. Skip any that don't apply, and say briefly why each one does or doesn't matter here:

- **Data:** migrations and their order, backfills, backward compatibility, and what happens to existing records.
- **Security:** authentication and authorisation on every new path, input validation at trust boundaries, and secrets.
- **Performance:** the hot paths at expected load, query patterns, N+1 queries, and payload size.
- **Failure:** what happens when each dependency is slow or down, retries, idempotency, and partial writes.
- **Observability:** how you'll know it's broken in production (logs, metrics, alerts).
- **Rollout:** feature flags, deploy order, a rollback plan, and whether existing users need anything.
- **Testing:** what gets tested at which level, and the critical paths that must have coverage.
- **Debt:** shortcuts taken on purpose, what each one costs, and when to pay it back.

Anything this sweep surfaces either becomes a decision (back to Phase 3) or is recorded under **Risks and Caveats**.

## How Detailed the Spec Is

- **Decide every contract:** schemas, API shapes, interfaces between components, event payloads, the files that change, and the run and test commands. The build must never have to guess these.
- **Leave internals to the build:** function bodies, private helpers and variable names. Don't write pseudo-code.
- **Scale the depth to the milestone.** A milestone with one one-way door might produce a two-page spec. A milestone that touches the data model and three services deserves the full treatment. Depth goes where reversal is expensive.
- **There is no fixed number of exchanges.** The review takes as long as the one-way doors need. Answers the user has already given count, so don't re-ask settled ground.

## Write `inter/m<N>-<slug>/spec.md`

Keep the draft updated throughout. Give every component its own heading, because `inter-build` cites them, and cross-reference PRD headings. Drop any section that doesn't apply.

```markdown
---
doc: spec
status: draft
---
# M<N>: <Name> — Spec

## Current State
How the system works today where this milestone lands, with file references. What helps, what fights us.

## Gap
What exists, what we extend, what we change, and what's new.

## Design Overview
The chosen shape in plain sentences, and the PRD core journey traced through it ("user does X → handled in `path` → stored in Y → sees Z").

## Decisions
### D1: <Decision> — decided | open
- Door: one-way | two-way
- Context: why this decision exists here.
- Options: each option with how it works here, its gains, and its caveats.
- Chosen: the option picked, and who chose it (the user, or a recommendation they accepted).
- Why: tied to the milestone goal and the product direction.
- Tradeoff accepted: what we give up.
- Revisit if: the condition that would reopen this decision.

## Components
### <Component> (new | changed)
What it does, what it talks to, which PRD heading it serves, and which existing code it reuses.

## Data Model
Schemas, migrations and their order, backfill, and backward compatibility.

## Interfaces
Routes, commands, signatures and event contracts, with request and response shapes.

## External Services
Each with its exact calls, auth, limits, cost, and a doc link. Mark which details were verified.

## Look and Feel
The PRD's direction translated into this stack's components and tokens.

## Files
An annotated tree of new and changed files.

## Risks and Caveats
Each with its likelihood, its impact, and the mitigation or fallback.

## Rollout
Flags, deploy and migration order, and the rollback plan.

## Testing Strategy
What is tested at which level, and the critical paths.

## Debt Taken
Each deliberate shortcut, with its cost and when to pay it back.

## Run and Verify
Exact commands to install, start, test, lint and build. `inter-build` runs these.

## Open Issues
Anything unverified, and which build slice should check it first.
```

## Review

Show the spec, leading with the decisions. Give honest feedback in three to five sentences. Does every acceptance criterion have a home? Is any one-way door still resting on an unverified assumption? What is the biggest remaining risk? Ask once: "Look good, or anything to change?" A clear yes sets `status: approved`. If any decision is still `open`, the spec can't be approved; name the decision and resolve it first.

## Hand Off

"Spec approved. Next is `inter-build`, which turns it into ordered slices. It will put the riskiest open issue first. A fresh conversation is recommended."
