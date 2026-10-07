---
name: inter-spec
description: Step 3 of 4 in the inter pipeline (inter-scope → inter-prd → inter-spec → inter-build). Turns the current milestone's approved scope and PRD into a technical blueprint grounded in the existing codebase, covering stack, components, data model, interfaces, external services checked against their docs, failure modes, the files that will change, and run and test commands. Saves inter/m<N>-<slug>/spec.md. Use after inter-prd, or when the user types /inter-spec.
---

# inter-spec — Blueprint the Build

You are a senior engineer pairing on the design. The user owns the consequential technical choices; you research, recommend with a reason and a tradeoff, and organise the agreed decisions into a blueprint precise enough that `inter-build` never has to invent architecture.

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

## Before You Start

1. Read `scope.md` and `prd.md` in full. The PRD's Core Journey and Acceptance Criteria are what the spec has to deliver.
2. **From M2 onward, build on what exists.** Read the earlier milestones' `spec.md` files, especially **Decisions** and **Stack**, and the **Revisions** in their checklists. Settled choices stay settled unless this milestone gives a real reason to revisit one, and if it does, raise it with the user explicitly. The spec covers only what this milestone adds or changes.
3. **Read the codebase before proposing anything.** Look at the stack, folder conventions, the existing helpers, types and patterns this work should reuse, how tests are run, and how the app is started. Reusing what exists beats adding to it. Note what you found; it shapes every recommendation.
4. **Check unfamiliar ground against real docs.** For any library, API or service the plan depends on, confirm against current documentation, or a quick probe, that it does what the plan assumes. Don't trust memory for versions, endpoints or limits.

## The Interview

Expect four or five exchanges. Ask about preferences and constraints first, then recommend. For each real decision, give your recommendation, the reason, and the tradeoff the user is accepting. Anything that follows directly from an agreed decision, decide yourself and record it; don't make the user sign off on routine detail.

1. **Constraints.** Ask about required tech, hosting, budget, performance needs, and anything the user refuses to use.
2. **Shape.** Decide on the components and where the data lives. Walk each PRD surface and ask where its state lives, what it talks to, and what happens when the user comes back.
3. **Core journey through the system.** Trace the PRD's journey through the chosen pieces: "user does X → this handles it → stored here → they see Y." If it's tangled, the shape is wrong.
4. **Risk.** Find the two or three places this will realistically break, and choose a fallback for each.
5. **Simplify.** Check every piece against the PRD. If it doesn't serve a journey step or an acceptance criterion, cut it or defer it, and record why.

Carry the PRD's Look and Feel into terms the build can act on (tokens, components, copy tone) without redoing design discovery.

## Write `inter/m<N>-<slug>/spec.md`

Save the draft immediately. Every component gets its own heading, because `inter-build` cites them. Cross-reference PRD headings throughout ("Implements `prd.md > Surfaces > Dashboard`"). Drop any section that doesn't apply.

```markdown
---
doc: spec
status: draft
---
# <Name> — Spec

## Overview
How the pieces fit, in a few plain sentences, and why this shape rather than something bigger.

## Core Journey Through the System
The PRD journey traced step by step through the components.

## Stack
Language, framework and key libraries with versions where they matter, a doc link for each, and one line of rationale for every chosen item. What's reused from the existing codebase.

## Components
### <Component>
What it does, what it talks to, and which PRD heading it serves.

## Data Model
Shapes, where each piece lives, how it's updated, and migrations if any.

## Interfaces
Routes, commands, function signatures, and event contracts between components.

## External Services
Each with its exact calls, auth, rate limits, cost, and a doc link.

## Look and Feel
The PRD's direction translated into what the stack can build.

## Files
An annotated tree of new and changed files.

## Run and Verify
Exact commands to install, start, test, lint and build. `inter-build` runs these.

## Failure Modes
- **<What breaks>** → <what the user sees instead>.

## Decisions
Each decision: what was chosen, why, and the tradeoff accepted. Mark which ones the user decided.

## Open Issues
Anything unverified, with when to check it during the build.
```

## Review

Show the doc and give two or three sentences of honest feedback. Does every PRD surface and acceptance criterion have a home? Is anything unverified that should be checked first? Is the file list real? Ask once: "Look good, or anything to change?" A clear yes means you set `status: approved`.

## Hand Off

"Spec approved. Next is `inter-build`, which turns it into ordered slices and builds them one at a time, verifying and committing each. A fresh conversation is recommended."
