# Reference: Anti-Scroll Policy (overlays fit the viewport)

Panels that open *over* the screen — modals, dialogs, side drawers, and content-rich popovers — must fit
entirely within the viewport so the user sees all of their content and actions at once, without scroll-hunting.
When the confirm/cancel buttons of a modal sit below the fold of the overlay, a real user hesitates, scrolls
to look for them, or abandons the task. That is an **operability** failure, not a styling preference — the
same class of gap as a missing focus indicator. This lens audits *whether the actions and key content are
reachable without scrolling*, never the panel's padding, spacing, or colors (that is the visual pass, out of
scope).

> In scope: actions/content reachable without scroll-hunting; scroll contained to an internal area with a
> pinned header/footer. Out of scope: the panel's spacing, padding, radius, or color (visual/UI).

**Why this is in scope (the backing principles)**
- **Material Design — dialogs.** Most dialog content should avoid scrolling; when scrolling *is* necessary,
  the title pins to the top and the action buttons pin to the bottom so they stay visible while the body
  scrolls. The fixed-header/footer + internal-scroll pattern is the canonical fix.
- **Apple HIG — sheets, popovers, action sheets.** Avoid enabling scrolling in an action sheet; size a
  popover/sheet only big enough for its content; it shouldn't take over the screen.
- **NN/g — Overuse of Overlays.** If the content is too long to fit the overlay and forces a lot of
  scrolling, you are wasting screen space and should reconsider the overlay (or move to a full view).
- **Nielsen — "Scrolling and Attention."** Attention drops sharply below the fold; the *Illusion of
  Completeness* means users often believe a panel is complete and never scroll to the hidden actions.
- **WCAG 2.1 SC 1.4.10 Reflow.** Content and functionality must be available without two-dimensional
  scrolling; keep the controls needed for the task in view.
- **Fitts's Law** (see `interaction-laws.md`). A footer pinned to the panel edge is a large, stable,
  easy-to-hit target; an action floating at the end of a scroll is not.

Applies especially to: action modals (reassignment, confirmation, quick forms), side drawers, and floating
panels/popovers with structured content.

---

## The shape of a viewport-safe overlay

The container is height-capped to the viewport and never scrolls as a whole; only an inner body scrolls,
between a pinned header and a pinned footer that holds the actions.

```tsx
// Container: capped to the viewport, never scrolls as a whole
<div className="flex flex-col max-h-[90vh] overflow-hidden rounded-xl">
  <div className="flex-shrink-0 ...">{/* Pinned header */}</div>
  <div className="flex-1 min-h-0 overflow-y-auto ...">{/* Body — the ONLY scroll area */}</div>
  <div className="flex-shrink-0 ...">{/* Pinned footer with the actions */}</div>
</div>
```

For a modal with two independent sections (e.g. pick an area, then pick a user), lay them **side by side** to
cut the height needed instead of stacking them vertically. This two-column layout — `flex-row`, pinned
header/footer (`flex-shrink-0`), and `min-w-0` on the flexible column to stop horizontal overflow — is the
canonical shape for a multi-section action modal such as a reassignment dialog.

```tsx
// Multi-section modal: independent areas side by side to cut height, not stacked
<div className="flex flex-row flex-1 min-h-0 overflow-hidden">
  <div className="w-52 flex-shrink-0 overflow-y-auto ...">{/* Left panel (e.g. pick an area) */}</div>
  <div className="flex-1 min-w-0 overflow-y-auto ...">{/* Right panel (e.g. pick a user) — min-w-0 stops overflow */}</div>
</div>
```

These are CSS/layout examples, but the audit target is the *behavior* they produce: actions and content
reachable without scrolling. Frameworks other than React/Tailwind express the same shape with the same
properties (`max-height`, `overflow`, a flex/grid body that takes the remaining space).

---

## Detection signals

```
→ overlay container (modal/dialog/drawer/popover) with no max-height bound (no max-h-[Nvh] / maxHeight),
   rendering variable-length content (a mapped list) → the panel grows past the viewport and the footer is
   pushed below the fold → S3 (actions unreachable without scroll-hunting); S2 if only secondary info is cut
→ overflow-y-auto / overflow:auto on the OUTERMOST panel container (the whole panel scrolls) instead of an
   inner body → the header + actions scroll away with the content → S3
   Fix: container = flex flex-col max-h-[90vh] overflow-hidden; body = flex-1 min-h-0 overflow-y-auto;
        header/footer = flex-shrink-0
→ confirm/cancel/submit buttons rendered as the last children of a scrollable area (inside an overflow
   ancestor), not in a flex-shrink-0 footer → the user must scroll to find/confirm the action → S3
   (also H1 visibility of status, H3 emergency exit)
→ fixed pixel height on a content-dependent body (h-[400px] / height:400px) → overflows on short viewports,
   wastes space on tall ones → S2
   Fix: flex-1 min-h-0 overflow-y-auto instead of a fixed height
→ multi-section modal stacking two+ independent areas vertically (e.g. area picker + user picker) so the
   total height exceeds the viewport → S2
   Fix: lay independent sections side by side (flex-row) to cut the required height
→ flex-1 / flexible column with long content but no min-w-0 → text/list forces the panel wider or adds a
   horizontal scrollbar (two-dimensional scroll — WCAG 1.4.10) → S2
→ side drawer / bottom sheet with no max-height and no internal scroll → on short viewports its footer
   actions render off-screen with no way to reach them → S3
```

Severity note: escalate an unreachable action to **S4** when it blocks task completion on common viewports —
e.g. the *only* submit/confirm of a committed or irreversible flow sits below the overlay fold with no other
path. A panel that merely *could* be tighter, but whose actions are already visible, is not a finding here.

---

## What NOT to flag (scope reminder)

This lens is about *reachability without scrolling*, not layout aesthetics. Do **not** flag: the amount of
padding/margins in the panel, its width as a matter of taste, corner radius, shadow/elevation, or "this modal
feels cramped/roomy." Those belong to the visual/UI pass the user has already done. A correctly contained,
viewport-fitting modal with tight spacing is **not** a finding; an unbounded modal whose Save button is two
scrolls down **is**. If your note is "make it prettier," drop it; if it is "the user can't reach the action
without scrolling," keep it.

---

## How to use this lens

For every overlay you find (modal, dialog, drawer, content popover), simulate a short viewport (e.g. a laptop
at ~768px tall, or a phone): does the whole panel fit, or does something — especially the primary/confirm
action — fall below the fold? Confirm the scroll is contained to an inner body with a pinned header and a
pinned footer, that no container has an unbounded or fixed-pixel height, and that no flexible column overflows
horizontally. Each "no" is a finding with the behavioral fix above — never a restyle.
