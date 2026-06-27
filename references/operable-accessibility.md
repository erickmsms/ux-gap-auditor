# Reference: Operable Accessibility (a11y as usability)

Accessibility split in two: the **visual** half (color contrast, text size) is OUT of scope here — the user
has the visual layer handled. The **operability** half — can a person actually *operate* the product with a
keyboard, screen reader, or other assistive tech — is core UX and very much in scope. A keyboard user who
can't see focus, or a screen-reader user who hits an unlabeled button, simply cannot use the product. That's
a usability failure, not a styling preference.

> Out of scope here: contrast ratios, color choices, font sizes. In scope: everything below.

---

## Keyboard operability

The baseline: every interactive thing must be reachable and activatable by keyboard alone, in a logical
order, with a visible focus indicator and no traps.

**Detection signals**
```
→ click handler on a non-interactive element with no keyboard equivalent:
   <div onClick={...}>  with no role, tabIndex, or onKeyDown → S3 (keyboard users can't trigger it)
→ custom control (dropdown, tab, accordion, modal) built from <div>/<span> with no role + no key handling
   → S3
→ focus trap: a modal that doesn't return focus / lets Tab escape behind it → S3
→ positive tabindex (tabindex="3") forcing an unnatural order → S2
→ Escape doesn't close an open modal/menu → S2 (also H3a)
→ skip-to-content link absent on content-heavy pages → S2
```
Fix: use native interactive elements (`<button>`, `<a>`, `<input>`) wherever possible; if you must use a
div, add `role`, `tabIndex={0}`, and key handlers; trap & restore focus in modals; never positive tabindex.

---

## Focus visibility (operability, not aesthetics)

```
→ global outline removal with no replacement:
   *:focus { outline: none }  or  :focus-visible { outline: none }  → S3 (keyboard users lose their place)
→ focus shown only via a color change (no outline/ring) → S2 (lost in Windows High Contrast Mode)
```
Fix: keep a visible focus indicator — `:focus-visible { outline: 2px solid …; outline-offset: 2px }`.
Removing the default outline is fine ONLY if a clearly visible alternative replaces it. (This is about
*presence* of the indicator, not its color — so it stays in scope even though the visual layer is "done".)

---

## Screen-reader semantics (recognition for non-visual users)

```
→ icon-only button with no accessible name:
   <button><TrashIcon/></button>  with no aria-label → S3 (announced as "button", meaningless)
→ <img> conveying meaning with no alt; decorative <img> missing alt="" → S2
→ form input with no associated <label>/aria-label (placeholder ≠ label) → S3 (also H6b)
→ dynamic error injected with no role="alert"/aria-live → S2 (screen reader never hears it)
→ toast/status region with no aria-live="polite"/role="status" → S2
→ heading levels skipped or non-semantic (<div class="title">) → S2 (breaks navigation by heading)
→ decorative icon inside a labeled control missing aria-hidden="true" → S1 (double announcement)
→ <input aria-invalid="true"> missing on a field showing an error → S1
→ aria-describedby not linking helper/error text to its field → S2
```
Fix: every control has an accessible name; meaningful images have alt, decorative ones `alt=""`/`aria-hidden`;
dynamic feedback uses live regions; headings are real and ordered; errors set `aria-invalid` + link via
`aria-describedby`.

---

## Motion & input modality

```
→ animation/auto-advancing carousel with no prefers-reduced-motion guard → S2 (vestibular discomfort,
   and it's a control issue: the user can't stop it)
→ essential info delivered only on hover (no focus/tap equivalent) → S2 (touch & keyboard users miss it)
→ drag-and-drop as the ONLY way to perform an action (no keyboard/button alternative) → S3
→ time-limited action with no way to extend/pause → S2
```
Fix: respect `prefers-reduced-motion`; never hover-only for essential content; always provide a
non-drag alternative; let users extend or disable time limits.

---

## A practical testing ladder (recommend to the team)

Code review catches a lot, but real validation needs a quick manual pass per key flow:
1. **Keyboard only** — Tab through every flow; can you reach and activate everything, see focus, and escape
   modals? (catches the most common operability gaps)
2. **Screen reader** — VoiceOver / NVDA walkthrough; is every control announced meaningfully?
3. **Zoom 200%** and **reduced-motion** — does anything become unusable or unstoppable?

Automated tools (axe, Lighthouse) catch ~30–40% of issues — necessary but not sufficient. Frame these as
operability checks tied to real tasks, not a compliance checklist.

---

## Scope reminder
Flag operability and semantics. Do **not** flag contrast ratios, color, or text size — those belong to the
visual/UI pass the user has already completed. If a finding is really "this color is hard to read," leave
it out; if it's "a keyboard user can't reach or perceive this control," include it.
