# inter

Four agent skills for building a great product one step at a time: plan before you build, then build in verified slices. They work in any harness that reads `SKILL.md` files (Claude Code, Codex, Cursor and others).

## Install

```
npx skills add KshitijBhardwaj18/specbuilder --all -y
```

## The four steps

| Skill | What happens | Output |
|---|---|---|
| `inter-scope` | Interviews the idea down to its kernel, who it's for, what done looks like, and what's in now, later or cut | `inter/<slug>/scope.md` |
| `inter-prd` | Defines what it does: core journey, surfaces, behaviour, edge cases, look and feel, acceptance criteria. No code talk. Can be skipped for work with no user-facing surface | `inter/<slug>/prd.md` |
| `inter-spec` | Defines how it's built, grounded in the existing codebase: stack, components, data, interfaces, failure modes, files, and run/test commands | `inter/<slug>/spec.md` |
| `inter-build` | Turns the plan into ordered end-to-end slices, then builds, verifies, commits and ticks each one. Ends with a hands-on review, a secrets check, a push and a PR | `inter/<slug>/checklist.md` plus the code |

You run each step by invoking its skill. Starting a fresh conversation between steps is fine, because the files carry the context.

## How state works

The `inter/` folder at the repo root is the only state. Each feature gets its own folder under `inter/<slug>/`, and there is no separate progress file.

- Every doc has `status: draft | approved` in its frontmatter. A PRD can also be `skipped`, and the checklist becomes `done` once the PR is open.
- Each skill saves a draft as soon as one exists, and marks it `approved` when you say it looks good. There is no second sign-off.
- `checklist.md` tracks build progress through its `- [ ]` boxes. Plan changes made during the build are recorded under `## Revisions`.

Every skill starts by reading `inter/`, telling you where the feature stands, and sending you to the right step. That means you can invoke any of them at any point.

## Layout

```
skills/<name>/SKILL.md    each skill, with its doc template inline
```
