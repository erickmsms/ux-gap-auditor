# Reference: Usability Heuristics (with code-level detection)

Nielsen's 10 heuristics are the field-standard lens for usability problems. This file gives each one a
**code detection signal** (what to grep/read for in a repo) plus the fix. Audit *behavior*, not styling.

Severity reminder (Nielsen 0–4): 4 catastrophe · 3 major · 2 minor · 1 cosmetic.

---

## H1 — Visibility of system status
*Users should always know what's going on, via timely feedback.* Silence after an action is the #1 cause of
double-submission and abandonment.

**H1a — Async action with no in-flight feedback**
```
Signal: onClick/onSubmit → await fetch/axios/api/supabase, but the trigger element is never
        disabled and no loading state is set while the promise is pending.
  <button onClick={handleSave}>Save</button>
  async function handleSave(){ await api.post(...) }   // nothing toggles during the await → S4 on
                                                        // payments/irreversible, S3 otherwise
Fix: disable the control and show progress for the duration of the request:
  <button disabled={isLoading}>{isLoading ? <Spinner/> : 'Save'}</button>
Exempt: static navigation / modal-open buttons (no async work).
```

**H1b — No success confirmation after submit**
```
Signal: submit handler resolves with no toast / redirect / state change / inline confirmation.
  await api.post('/contact', data)   // then nothing → S3
Fix: confirm every intentional submit — toast for minor, confirmation screen for significant
     (order placed, account created). Real-time/autosave fields are exempt per keystroke.
```

**H1c — No progress indicator on multi-step flows**
```
Signal: component holds step/currentStep state for a 3+ step flow but renders no step indicator.
  const [step,setStep]=useState(1)   // no "Step 2 of 4" / progress bar → S2
Fix: any flow with 3+ steps shows where the user is and how much remains.
```

**H1d — Background work with no system status**
```
Signal: long operations (upload, export, sync) with only a spinner and no progress/percentage, or
        polling with no "syncing…" indicator. Operations > 10s with no progress → S3.
Fix: determinate progress when measurable; status text ("Uploading 3 of 8…") otherwise.
```

---

## H2 — Match between system and the real world
*Speak the user's language, not the system's.*

**H2a — Technical jargon / raw errors in user-facing copy**
```
Signal: user-facing strings containing raw codes or dev terms:
  "Error 404" / "500 Internal Server Error" / "ECONNREFUSED" / "null" / "undefined" / "NaN"
  "Invalid payload" / "Connection timeout" / stack traces / unresolved i18n keys ("error.auth.token") → S3
Fix: translate to human language — "We can't find that page", "Can't connect — check your internet."
Exempt: developer tools, API docs, admin/debug panels (that audience expects it).
```

**H2b — Icon whose conventional meaning doesn't match its function**
```
Signal: <TrashIcon/> used for "Archive"; <SearchIcon/> in a "Zoom" button; aria-label contradicts the
        icon component name. → S2
Fix: use the icon that matches the real-world mental model, or pair an ambiguous icon with a text label.
```

---

## H3 — User control and freedom
*Users need a clearly marked emergency exit.* Feeling trapped destroys trust.

**H3a — Modal/dialog with no close mechanism**
```
Signal: role="dialog" / <dialog> with no close (×) button and no Cancel CTA → S3;
        no Escape-key handler → S2; no backdrop-dismiss when it should be dismissible → S2.
Fix: every modal has a visible close + responds to Escape. Intentionally uncloseable modals
     (session expired) still need one clear next action.
```

**H3b — Multi-step flow with no way back / no exit**
```
Signal: step > 1 rendered with no onBack/setStep(step-1) → S2; a committed flow (checkout) that can
        only be left by completing it → S4.
Fix: every step past the first has Back; long flows add Cancel or Save & Exit. If a step legitimately
     can't go back (payment processing), disable Back with an explanation rather than hiding it.
```

**H3c — Destructive action with no confirmation / undo**
```
Signal: onClick={()=>deleteItem(id)} with no confirmation dialog and no undo → S2 (reversible),
        S4 (permanent + unconfirmed). Soft-delete with no "Undo" toast → S2.
Fix: permanent destructive actions get a confirmation that NAMES the item ("Delete 'Project Alpha'?");
     reversible ones get a 3–5s Undo toast. Never a generic "Are you sure?".
```

---

## H4 — Consistency and standards
*Same thing, same word, same place.* Inconsistency forces relearning.

```
Signal: the same concept labeled differently across the app (workspace/project/team;
        delete/remove/archive); the same action triggered by different controls in different places;
        platform conventions ignored (back gesture, native pickers). → S2
Fix: one word per concept (keep a glossary); reuse the same control for the same action everywhere.
```

---

## H5 — Error prevention
*Prevent the mistake before it needs a message.*

```
Signal: free-text input where a constrained control fits (date typed as text vs date picker);
        destructive control adjacent to a common one with identical weight; no inline validation before
        an expensive submit; no auto-save on a long form (data-loss risk). → S3 for data-loss risks
Fix: constrain inputs, separate destructive actions, validate inline, auto-save long forms.
```

---

## H6 — Recognition rather than recall
*Make options visible; don't make users remember.*

**H6a — Icon-only primary navigation, no labels**
```
Signal: <nav> items with an icon child but no visible <span> label → S2 (desktop sidebar/top nav);
        S1 on mobile bottom nav (space-constrained). aria-label alone is accessible but still a recall
        burden for sighted users.
Fix: primary nav shows text labels next to icons; tooltips are fine for secondary/toolbar actions only.
```

**H6b — Label used as placeholder (disappears on type)**
```
Signal: input with placeholder and no persistent <label>; CSS that hides the label on focus.
  → S3 (no label at all) / S2 (label obscured during input)
Fix: a persistently visible label above the field. Material "float to top" is fine if it stays visible.
```

**H6c — Selection not reflected back**
```
Signal: dropdown trigger still shows placeholder text after a value is chosen → S3; open list marks no
        selected option (no check/highlight) → S2; active filters indistinguishable from inactive → S2.
Fix: triggers always show the current value; selected/active options are clearly marked.
```

---

## H7 — Flexibility and efficiency of use
*Accelerators for experts, without burdening novices.* Most relevant for productivity tools, dashboards,
admin UIs; rarely needed in simple consumer apps.

```
Signal: complex/frequent actions (save, search, new) with no keyboard shortcut → S1/S2 on power-user
        tools; management list (inbox, files, tasks, orders, 10+ items) with no bulk-select → S2;
        user settings/preferences not persisted across sessions → S2.
Fix: add shortcut hints ("Save ⌘S"), bulk selection on management lists, and persist preferences.
```

---

## H8 — Minimalist content (NOT visual minimalism)
*Every extra unit of content competes with the relevant unit.* Audit **information clutter**, not looks.

```
Signal: dialogs/forms asking for more than the task needs right now; screens dense with rarely-used
        options shown at the same level as primary ones; walls of help text on simple fields. → S2
Fix: defer non-essential fields/options (progressive disclosure); surface the primary action, tuck the
     rest into overflow/advanced. (Do not comment on spacing, font size, or color — that's UI.)
```

---

## H9 — Help users recognize, diagnose, and recover from errors
*Errors must be in plain language and offer a way out.*

```
Signal: error string states the problem but no recovery step ("Invalid email") → S2;
        error with no information at all ("Error", "Invalid input") → S3;
        form that clears user input on error → S4 (forces full re-entry).
Fix: every error says what's wrong AND how to fix it; preserve all entered input; one-click retry for
     transient failures. See references/microcopy.md for the message formula.
```

---

## H10 — Help and documentation
*Discoverable, task-focused, concrete.*

```
Signal: format-constrained fields (password rules, phone, VAT, file type/size) with no helper text → S2;
        complex/technical settings (webhook, CRON, permissions) with no inline help → S2;
        empty state / first-run with no next-step guidance → S2 (complex product) / S1 (simple).
Fix: show requirements before submit (not only in the error); add contextual help to genuinely ambiguous
     fields; every empty state tells the user what it's for and how to fill it.
```

---

## Quick checklist
- [ ] Every async trigger has a loading/disabled state (H1a)
- [ ] Every intentional submit confirms success (H1b)
- [ ] 3+ step flows show progress and allow Back/Cancel (H1c, H3b)
- [ ] No raw error codes / dev jargon in user copy (H2a, H9)
- [ ] Every modal has a close + Escape (H3a)
- [ ] Destructive actions confirm and/or offer undo (H3c)
- [ ] One word per concept across the app (H4)
- [ ] Inputs constrained to prevent errors; long forms auto-save (H5)
- [ ] Labels persist; selections reflected back (H6)
- [ ] Error messages: what's wrong + how to fix; input preserved (H9)
- [ ] Constrained/technical fields and empty states have guidance (H10)
