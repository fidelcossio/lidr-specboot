---
name: preview
description: Use when an OpenSpec change has visual decisions still open (layout, rows, buttons, badges, number formats, colors, density) and the user wants to see and refine them before /opsx:apply. Triggers - /preview, "show me a preview", "mock it in HTML", "is it worth previewing before apply?", or UI proposals that are written but not applied.
version: 1.0.0-cashflow
---
# preview

## Overview

Fixing a UI decision after `/opsx:apply` costs edits across many files, a browser verification and a deploy. Fixing it in one static HTML file costs a few small edits. This skill turns a proposed UI change into a throwaway preview, iterates on it with the user, and **only after explicit acceptance** writes the decisions back into the OpenSpec artifacts. It never touches app code.

**Core rule:** preview first, proposals second, code never. Violating the order is violating the skill.

## When to use

- One or more changes (e.g. R48, R49, R71) are proposed, not applied, and their risk is visual rather than logical.
- The user asks "does a preview make sense, or is it the same cost as apply?" Answer that first, before building anything. A preview pays off when: the uncertainty is how it looks, the apply touches many sites, or apply ends in a deploy. It does not pay off for pure logic, data or a single-value tweak (answer in chat instead).

Not for: logic or data-model changes, bug fixes, anything the preview cannot represent.

## Process

1. **Scope.** Get the change names or R-numbers. Read only what decides the look: `proposal.md` (Why), `design.md` (Decisions, Open Questions) and the requirement titles in the specs. Do not read whole artifacts.
2. **Build ONE HTML file** at `openspec/vistas-previas/<slug>.html`.
   - Load the **real** `design-system/tokens.css`, `bundle.css`, `bundle.js` and `app/styles.css`. Never redraw what exists. React from the CDN already used in `index.html`; no Babel, no app code, no Firebase.
   - Each demo is a **Before (today) / After** pair with realistic sample data from the inventory in the specs, in the app's language (Spanish here).
   - Toolbar: light/dark and mobile (375)/web toggles. Add `*{transition:none!important}` (the browser pane does not advance transitions in the background, so screenshots show half-painted colors).
   - Put each open question in a boxed note next to the demo it affects. Say plainly what is mock (components the system does not have yet) and draw missing icons inline, listing them as requests to the design system.
3. **Make it openable.** The built-in browser opens `file://` pages as `data:` snapshots, so relative CSS/JS paths break. Generate `<slug>.autonomo.html` with a small script (scratchpad) that inlines the CSS and JS, and deliver that file with `SendUserFile`. If `preview_start` is at its server limit, do not fight it.
4. **Verify before showing.** Open it, read console errors, check element counts. Check **dark mode** (tokens like `--page-soft` can equal the card and make fills vanish) and **375 and 1280 px** (`resize_window`; container queries need a wide viewport; reset it after). Prefer `getComputedStyle` over screenshots when colors look odd.
5. **Iterate.** Each round: apply the user's changes **to the HTML only**, regenerate the standalone file, resend it, and list what changed and what is still open. Do not edit proposals, PRD or code during iteration, even if the user's answer sounds final. Record answered questions in the page ("Decidido: …"). Use Python for edits, not `sed -i` (macOS), and re-open the page after each edit: one unbalanced parenthesis blanks it.
6. **Acceptance gate.** Proceed only on an explicit "update the proposals" or equivalent. A new decision after that goes back to step 5 first.
7. **Update the proposals.** Run `/opsx:update <change>` for each affected change; the user's explicit go-ahead covers the agreed content.
   - Revise proposal, design, spec and tasks together; add decisions as numbered design decisions, update scenarios and the verification tasks.
   - Grep the **other** open changes for the same components and reconcile contradictions (a decision in one change often invalidates another).
   - Defects the preview exposed in the design system go to its "requests" list, not to a local patch.
   - Run `openspec validate <change> --strict` on every touched change.
8. **Close.** Report which artifacts changed, decisions still open, and work no proposal covers yet. Point to `/opsx:apply`; never run it.

## Common mistakes

| Mistake | Fix |
|---|---|
| Updating proposals as soon as the user sounds happy | Wait for the explicit go-ahead (step 6) |
| Rebuilding buttons/rows by hand | Load the real CSS and components; the preview must show what the system really does |
| Judging dark mode from a mid-transition screenshot | Disable transitions; read computed colors |
| Adding lists or screens the user did not name | Keep scope; list the rest as "pending, apart" |
| Presenting a mock as the spec | State what is mock and what the proposal will require |
| Fixing one change and forgetting its neighbours | Step 7 grep across open changes |
| Starting a dev server for a static file | Use the standalone `.autonomo.html` |
