---
name: ui-discipline
description: >
  Frontend component architecture, state discipline, and accessibility. Use this skill whenever
  the task builds or restructures UI — components, pages, forms, design-system pieces — in any
  framework. Also trigger proactively when reviewing UI code that mixes data fetching with
  rendering, duplicates state, hand-rolls interactive controls out of divs, or ships without
  keyboard/screen-reader support. Framework-neutral: the rules below apply whether the stack is
  React, Vue, Svelte, or server-rendered templates.
---

# UI Discipline

You are building interface code that stays maintainable and usable by everyone. Two failure
modes dominate frontend rot: state scattered until no one knows what owns the truth, and
interactivity bolted onto markup that only works for a mouse user. This skill prevents both.

---

## Component Architecture

- **One job per component.** A component either orchestrates (fetches data, owns state, composes
  children) or presents (renders props, emits events) — mixing both in one unit is the primary
  source of untestable UI. Keep presentational components pure: same props, same output.
- **Compose, don't configure.** When a component sprouts its fifth boolean prop, split it or
  accept children/slots instead. Prop explosions are inheritance hierarchies in disguise.
- **Match the system.** Reuse the project's existing components, tokens, and spacing scale
  before writing new ones. A visually "better" one-off widget is a net loss to consistency.
- **Colocate.** Styles, tests, and helpers live with the component they serve, not in distant
  global buckets.

---

## State Discipline

- **Single source of truth.** Every piece of state has exactly one owner; everything else
  derives from it at render time. If two states can disagree (`items` and `itemCount`),
  one of them shouldn't exist.
- **Lift state only to the lowest common owner** — no higher. Global stores are for genuinely
  global concerns (session, theme), not a dumping ground for prop-drilling fatigue.
- **Server state is not UI state.** Remote data comes with loading, error, empty, and stale
  states — model all four explicitly. Every fetch renders something sensible in each state;
  the unhandled error state that renders a blank screen is a bug, not an edge case.
- **Forms:** validate on submit, show errors next to the field they belong to, preserve user
  input through failures, and disable double-submits.

---

## Accessibility — non-negotiable baseline

Work through this on every interactive surface:

1. **Semantic elements first.** `button` for actions, `a` for navigation, `label` wired to
   every input, headings in order, landmarks for page regions. ARIA is the fallback for
   things HTML can't express — not a patch for divs pretending to be buttons.
2. **Keyboard complete.** Every action reachable by keyboard alone: logical tab order, visible
   focus ring (never `outline: none` without a replacement), Escape closes overlays, focus
   moves into opened dialogs and returns on close.
3. **Screen-reader coherent.** Images get meaningful `alt` (or empty for decorative); icon-only
   buttons get accessible names; dynamic updates that matter get announced (live regions).
4. **Contrast and target size.** Text contrast ≥ 4.5:1 (3:1 for large text); touch targets
   comfortably tappable; information never conveyed by colour alone.
5. **Respect user settings.** Honour `prefers-reduced-motion` and `prefers-color-scheme`;
   layout survives 200% zoom and text resizing (relative units, no fixed-height text boxes).

---

## Rendering & Robustness

- Design mobile-first and let the layout grow; wide content (tables, code) scrolls in its own
  container — the page never scrolls horizontally.
- Prevent layout shift: reserve space for images/embeds with explicit dimensions.
- Escape/sanitise anything user-generated before it hits the DOM (`/secure-coding-practices`
  covers the full XSS posture).
- Keep main-thread work during interaction light; defer what isn't needed for first paint
  (`/perf-tuning` when a Core Web Vitals budget is in play).

---

## Verification

1. Suite green, plus tests for the component's states: default, loading, error, empty.
2. Keyboard-only pass: complete every flow without touching the pointer.
3. Check the rendered output in a real browser at mobile and desktop widths, light and dark
   (`/browser-verify` when a browser tool is available).
4. Run an accessibility audit (axe/Lighthouse) — zero new violations.
