---
status: completed
experience: none
---

# Separate selection and project properties

## Goal

Reduce the Properties panel's first-screen density by keeping selection-scoped
editing in the default view and moving project-wide placement and diagnostics
workflows into a deliberately selected Project view. Compact repeated,
read-only component metadata and direct wire actions without removing the
existing semantic controls.

## State and Ownership

Start state from `git status --short --branch`:

```text
## codex/properties-project-split
```

This worktree is clean. The source worktree's untracked `.pnpm-store/` and
`.worktrees/` directories are local infrastructure and are not present here;
they are outside this target.

Owned paths:

- `apps/editor/src/app/App.tsx`
- `apps/editor/src/styles.css`
- `apps/editor/src/features/selection/selection-inspector-details.tsx` when a
  small diagnostics presentation change is required
- focused editor tests affected by the new view switch
- `plan/2026-08-23-properties-project-split/plan.md` and `plan/log.md`

Read-only boundaries:

- The MOS Bulk section in `App.tsx`, including its complete current content,
  order, actions, and NMOS/PMOS defaults. It must not be edited, moved, or
  collapsed.
- Circuit-model, placement, diagnostics, and netlist contracts; this target is
  presentation and view composition only.

Shared dependencies:

- `ProjectDiagnosticsSection` preserves its severity-only filtering and
  locator-backed selection behavior.
- Existing selection, placement-tray, and route action callbacks remain the
  interaction authority.

## Work

1. Add a compact Selection / Project entry point to Properties. Keep selected
   object editing, including MOS Bulk, in Selection; render Placement Tray and
   project diagnostics only in Project.
2. Compress repeated component metadata without losing semantic fields:
   suppress empty primitive targets and empty advanced parameters, tighten
   identity/terminal presentation, and retain visible coordinate and rotation
   controls plus the value/reference visibility row.
3. Keep route commands direct and selection-scoped, with conditional actions
   shown only when applicable; do not add a route overflow menu.
4. Add focused coverage for the view separation and validate the editor's
   existing interaction surface.

## Validation

- `pnpm test:local <focused test paths>`
- `pnpm test:e2e:local apps/editor/e2e/manual-editor.spec.ts --grep <relevant pattern>` when a stable relevant scenario exists
- `pnpm gate:affected -- --base origin/main`
- `git diff --check`
- `git status --short --branch`

## Gate Review

- Decision: affected
- Early gates: `pnpm gate:review:check -- --base origin/main`, `pnpm ci:static`, and `pnpm test:impact -- --base origin/main`
- Affected gates: `pnpm test:local` and `pnpm test:e2e:local apps/editor/e2e/manual-editor.spec.ts`
- Final gates: `pnpm ci:check` before any merge to `main`; this branch delivery also requires remote required checks.
- Platform risks: browser layout and interaction semantics require a local browser smoke check; no generated or release artifact is changed.

## Test Impact

- Decision: tests-updated
- Contracts: project-wide diagnostics and unplaced instances must be reachable
  without occupying the selection editing flow; existing direct property and
  route actions remain available.
- Primary checks: focused `App` or selection-inspector test plus the editor
  browser interaction spec selected from the real diff.

## Commit Intent

Commit as:

```text
refactor(editor): separate project properties from selection
```

## Outcome

Properties now has an explicit Selection / Project switch. Selection keeps
editing, direct wire actions, the visible value/reference row, placement
coordinates and rotation, and the unchanged MOS Bulk section. Project owns the
Placement Tray and live Issues. Primitive-only netlist target cards and empty
advanced parameter disclosures no longer consume initial space; the add action
opens the disclosure when needed. Diagnostics now filter solely by severity.

Validation passed: preflight; full affected unit suite (183 files / 1185
tests); component-insert (24), hierarchy (12), and manual-editor (98) browser
specs; isolated local-browser view checks; build; `git diff --check`.
