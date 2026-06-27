# Reference: States & Feedback coverage

Beginner UIs design only the happy path. In reality users spend large amounts of time in *other* states.
Each must be designed. This is the single richest source of UX gaps in a codebase — hunt the missing
branches.

---

## The state-coverage map

Every interactive surface can be in these states. For each component you audit, check which apply and
whether each is actually handled in code (not just visually present).

| State | When | How often forgotten |
|---|---|---|
| Default | nothing happened yet | always present |
| Loading | waiting on data/action | ⚠️ often missing |
| Empty | no data yet | ⚠️ often missing |
| Error | something failed | ⚠️ often missing |
| Success | action completed | ⚠️ often missing |
| Disabled | action unavailable | 🟡 sometimes missing |
| Focus | reached via keyboard | ⚠️ often missing (see operable-accessibility.md) |

A Button has no Empty state; a table has no Active state — use judgment and mark genuinely-N/A cells.

---

## Loading

**Pattern selection**
- Skeleton screen → content-heavy layouts (feeds, cards, tables); beats spinners for perceived speed.
- Spinner → short action-triggered waits (button click).
- Progress bar → long, measurable operations (upload, export).
- Optimistic UI → update immediately, reconcile/rollback on server response.

**Latency thresholds (see interaction-laws.md for the Doherty detail)**
- < 100ms: no indicator (a spinner flash is itself disruptive).
- 100ms–1s: subtle indicator / skeleton.
- 1–10s: clear loading state, progress if measurable.
- 10s+: detailed progress + estimate + background option.

**Detection signals**
```
→ data fetch (useQuery/useEffect+fetch/await) rendering content with no loading branch → S3
   {data && <List/>}  with no {isLoading && <Skeleton/>}  → blank screen during load
→ multiple competing spinners on one screen (5 components each spinning) → S2 (page feels broken)
→ skeleton shapes that don't match real content layout → S1
→ <section aria-busy="true"> missing during load → S1 (screen-reader status)
```
Rules: never a blank screen; show the indicator immediately; consolidate competing spinners; match
skeleton shapes to real content; respect `prefers-reduced-motion` for shimmer.

---

## Empty states

The first thing a new user sees, and often what decides whether they return.

**Anatomy:** icon/illustration · friendly headline ("You haven't added any tasks yet") · optional context ·
clear CTA ("Create your first task").

**Types (each needs distinct copy/CTA)**
| Type | CTA |
|---|---|
| First-use (new user) | Create first item |
| Cleared (deleted all) | Undo / create new |
| Search / filter (no match) | Clear filters / refine |
| Error-caused (load failed) | Retry |
| Permission (no access) | Request access |

**Detection signals**
```
→ {items.map(...)} with no {items.length === 0 && <EmptyState/>} fallback → S2
→ search/filter result with no distinct "no results for X" state (reusing first-use copy) → S2
→ empty state with no CTA → user stranded → S2
→ sad/broken imagery in empty states (implies the product failed) → S1
```

---

## Error states

**Levels & placement**
| Level | Handle with |
|---|---|
| Field validation | inline message below the field |
| Form-level | summary at top + highlight the offending fields |
| Action failure | toast / inline near the action, with retry |
| Page-level (404/500/offline) | designed full-page state with recovery, not the browser default |
| Partial failure | inline error on failed items, rest stays usable |

**Detection signals**
```
→ try/catch that swallows the error with no user-facing message (console.log only) → S3
→ submit handler with no .catch / error branch → user sees nothing on failure → S3
→ form that resets/clears fields on error (setState to empty, uncontrolled reset) → S4 (data loss)
→ page-level error rendering the raw error object or a blank screen → S3
→ no retry affordance after a transient failure → S2
→ error conveyed by color only (red border, no text/icon) → S2 (see operable-accessibility.md)
```
Rules: say what's wrong AND what to do; never blame the user; place the error near its cause; **preserve
user input**; make retry one click.

---

## Success states

Closing the loop prevents double-submits, refreshes, and "did it work?" anxiety.

| Pattern | Best for | Duration |
|---|---|---|
| Toast/snackbar | minor confirmations (Saved, Copied) | 3–5s auto-dismiss |
| Inline confirmation | form/settings saved | until next action |
| State change | button → "Saved ✓" | persistent |
| Full screen | payment done, account created | one-time, then redirect |

**Detection signals**
```
→ successful mutation with no confirmation of any kind → S3
→ toast dismissed < 2s → S2 (too fast to read / for screen readers to announce)
→ toast never auto-dismissed → S1 (blocks UI)
→ toast region missing aria-live="polite"/role="status" → S2 (screen readers miss it)
```
Rules: be specific ("Project 'Q4' saved"); don't use green for non-good news ("Account deleted"); make
toasts screen-reader accessible.

---

## Disabled states

```
→ control visually greyed with neither `disabled` nor `aria-disabled` → S3 (looks disabled, still fires)
→ disabled element removed from layout entirely → S2 (confusing — show it, make it inert)
→ no explanation why it's disabled → S1 (a tooltip "You need admin access" is far better)
→ pointer-events:none alone, no aria-disabled → S2 (keyboard can still activate)
```

---

## Feedback hierarchy (where to put the response)

Prefer feedback closest to the action:
1. Inline / contextual — at the element the user acted on (preferred).
2. Component-level — within the current component.
3. Page-level — banner or toast.
4. System-level — notification outside the current view.

A finding worth flagging: feedback that's too far from its cause (e.g. a field error shown only in a
top-of-page banner for a field at the bottom) → S2.
