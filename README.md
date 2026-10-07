# inter

Agent skills for building a great product one step at a time: plan before you build, then build in verified slices. They work in any harness that reads `SKILL.md` files (Claude Code, Codex, Cursor and others).

## Install

```
npx skills add KshitijBhardwaj18/specbuilder --all -y
```

## The steps

| Skill | What happens | Output |
|---|---|---|
| `inter-scope` | First run: interviews the idea down to its kernel and who it's for, then defines M1 only. Later runs: picks the next milestone from what shipped, what was learned and the ideas list | `inter/product.md`, `inter/m<N>-<slug>/scope.md` |
| `inter-prd` | Defines what the milestone does: core journey, surfaces, behaviour, edge cases, look and feel, acceptance criteria. No code talk. Can be skipped for work with no user-facing surface | `prd.md` |
| `inter-spec` | A staff-engineer design review. It maps the existing code where the milestone lands, ranks the real decisions (one-way vs two-way doors), and discusses each with options, caveats and a recommendation tied to the goal. Then it sweeps for data, security, performance, rollout and testing concerns. Every contract is decided; internals are left to the build | `spec.md` |
| `inter-build` | Turns the plan into ordered end-to-end slices, then builds, verifies, commits and ticks each one. Ends with a hands-on review, a secrets check and a PR. A separate agent then reviews each PR against the full plan, the agreed fixes are pushed, and `product.md` is updated | `checklist.md` plus the code |
| `inter-verify` *(optional)* | Drives the running app in a real browser through the core journey, every acceptance criterion, edge cases, and earlier milestones' journeys as regression, watching the console and network along the way. Failures are fixed in the same PR | `verify.md` |

You run each step by invoking its skill. Starting a fresh conversation between steps is fine, because the files carry the context.

## Milestones, one at a time

```
inter/
  product.md          kernel, who it's for, Shipped log, Ideas, Cut
  m1-core-loop/       scope.md  prd.md  spec.md  checklist.md
  m2-sharing/         ...
```

Only the next milestone is ever planned. Everything beyond it stays a loose, unordered list under **Ideas**. When a milestone ships, `inter-build` logs it under **Shipped** and adds anything the build taught you to **Ideas**. Then `inter-scope` helps you choose the next milestone with that knowledge.

M1 is the kernel plus the core loop: the smallest version that is genuinely good. Each later milestone builds on the existing code and on settled decisions instead of re-planning from scratch.

## How state works

The `inter/` folder at the repo root is the only state; there is no separate progress file.

- Milestone docs carry `status: draft | approved` in their frontmatter. A PRD can also be `skipped`, and a checklist becomes `done` once its PR is open.
- Each skill saves a draft as soon as one exists, and marks it `approved` when you say it looks good. There is no second sign-off.
- `checklist.md` tracks build progress through its `- [ ]` boxes. Plan changes made during the build are recorded under `## Revisions`.

Every skill starts by reading `inter/`, telling you where things stand, and sending you to the right step. A new idea that comes up mid-milestone goes into **Ideas** instead of derailing the current work.

## Layout

```
skills/<name>/SKILL.md    each skill, with its doc template inline
```
