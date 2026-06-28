# Reference: Microcopy & Content

Microcopy — button labels, errors, placeholders, confirmations, empty-state text — is usually written by
developers as an afterthought, yet it has outsized impact on usability and trust. This is **content
behavior**, not visual styling, so it is firmly in scope. When you flag an item, quote the exact text and
cite `file:line`.

---

## Button labels — the verb rule

Buttons should be a **verb (phrase)** describing the action.

| ❌ Vague | ✅ Specific |
|---|---|
| OK | Save / Confirm / Got it |
| Submit | Send message / Create account / Place order |
| Yes | Delete / Yes, cancel my plan |
| Cancel | Keep editing / Go back |

**Detection signals**
```
→ button text in {OK, Submit, Yes, No, Continue} with no object → S1/S2 (vague verb)
→ destructive button labeled only "Delete"/"Remove" (doesn't name the target) → S2
→ button that sends an email labeled "Confirm" (label ≠ outcome) → S2
```
Rules: name the outcome ("Save changes"); destructive buttons name the thing ("Delete project"); keep
primary CTAs 2–4 words.

**Confirmation dialogs** — name the thing, label the action:
```
Title:  Delete "Project Alpha"?
Body:   This permanently deletes the project and all its tasks. This can't be undone.
Buttons:[Cancel] [Delete project]      ← never [No] [Yes] under "Are you sure?"
```

---

## Error messages — the formula

A good error answers: **what went wrong → (why) → how to fix it.**

| ❌ Bad | ✅ Good |
|---|---|
| Invalid input | Please enter a valid email address |
| Error | We couldn't save your changes — check your connection and try again |
| Required field | Please add a project name to continue |
| 500 Internal Server Error | Something went wrong on our end — try again in a moment |
| File too large | This file is too large. Maximum size is 10MB |

**Detection signals**
```
→ error string < 4 words and non-actionable ("Invalid input", "Error") → S3
→ error states problem but gives no fix ("Invalid email") → S2
→ technical leakage in user copy (stack trace, status code, "null") → S3 (also H2a)
```

**Validation timing**
```
→ validating format on every keystroke ("Invalid email" while still typing) → S2 (annoying)
Rules: validate format on blur; password strength on change (live); required-field checks on submit.
```

---

## Placeholders are not labels

Good for: format hints (`name@example.com`, `YYYY-MM-DD`), short examples (`e.g. Project Alpha`).
Not for: replacing labels, long instructions, required info.

**Detection signals**
```
→ input with placeholder and no associated <label>/aria-label → S3 (label disappears on type — also H6b)
→ placeholder that merely repeats the label → S1 (no added value)
→ informal "eg:"/"E.g:" prefix instead of "e.g. " → S1
```

---

## Tooltips & helper text

- Tooltips: explain icon-only buttons, clarify a non-obvious field, explain why something's disabled.
  Not for essential info (put it on screen), not on mobile (no hover), not for long text.
- Helper text (static, below input): format hints, consequences ("Visible to your whole team"), counters
  (`0/280`). Position directly below the input, above where the error will appear.

```
→ icon-only button with no tooltip/aria-label → S2 (also operable-accessibility)
→ essential constraint hidden in a hover tooltip on mobile → S2
```

---

## Terminology consistency

One of the most common content gaps: the same concept named differently in different places.

| Concept | Don't mix |
|---|---|
| Workspace | workspace / project / team / organization |
| Content item | task / item / card / ticket |
| Saving | save / update / apply / confirm |
| Removing | delete / remove / archive / clear |
| People | members / users / collaborators |

**Detection:** grep the UI strings for synonyms of the same concept across components → flag mismatches → S2.
Rule: one word per concept; keep a short glossary; every label/button/confirmation uses it.

---

## Data formatting (dates, numbers, units)

Functional content includes the *values* shown, not just the labels. Ambiguous or raw formatting forces
users to decode or mis-read data — a match-the-real-world failure (H2), not a visual choice.

| ❌ Ambiguous / raw | ✅ Clear |
|---|---|
| 01/02/03 | Feb 1, 2003 (locale-aware) / "2 days ago" |
| 1234567 | 1,234,567 / 1.2M |
| 2026-06-28T14:00:00Z | "Today at 2:00 PM" |
| 4500 (cents? dollars?) | $45.00 |

**Detection signals**
```
→ date in a numeric-only format with cross-region ambiguity (01/02/03 — is it Jan 2 or Feb 1?) and no
   explicit month / locale-aware formatting → S2
→ raw ISO timestamp or epoch shown to the user instead of a friendly or relative date → S2
→ large number with no thousand separators or abbreviation (1234567) → S1 (hard to scan — also Miller)
→ currency / measurement shown with no unit, or as raw minor units (4500 meaning $45.00) → S2
→ relative time ("2 hours ago") with no absolute timestamp on hover/title when precision matters → S1
```
Rules: prefer locale-aware or relative dates; separate or abbreviate large numbers; always show the unit;
pair a relative time with an absolute one (tooltip/`title`) when exactness matters. (Number chunking →
`interaction-laws.md`, Miller's law.)

---

## Confirmation & success copy

| Action | ✅ Confirmation |
|---|---|
| Saved | "Changes saved" / "Project saved" |
| Sent | "Message sent to Jamie" |
| Deleted | "Task deleted" + Undo |
| Copied | "Link copied to clipboard" |
| Invited | "Invitation sent to alex@example.com" |

Rules: name the thing acted on; offer undo for destructive/hard-to-reverse actions; don't over-celebrate
routine actions ("🎉 Amazing, you saved a file!" is patronizing).

---

## Per-role text audit guide

Classify each user-facing string by role before judging it:

| Role | Key rule |
|---|---|
| CTA button | a verb; names the outcome |
| Label | visible, not placeholder; required marked with legend |
| Placeholder | format hint only |
| Error | what / why / how; no blame; no jargon |
| Empty state | explains the state + gives a next action |
| Confirmation/toast | names the thing; offers undo if destructive |

Citation format: `🟡 Button "Submit" (Form.tsx:42) — vague verb. Use "Send report".`
