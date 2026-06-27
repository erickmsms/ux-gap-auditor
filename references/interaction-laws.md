# Reference: Interaction cost (the UX laws)

Four well-established laws describe the *cost* of an interaction: decisions, motor effort, memory, and
perceived time. Use them to spot friction that isn't a missing state but a too-expensive interaction. These
are behavioral, not visual — sizing a tap target is about *reachability/operability*, not aesthetics.

---

## Hick's Law — decision cost
*Decision time grows with the number and ambiguity of choices.* `RT = a + b·log₂(n+1)`.

The goal is fewer *simultaneous* choices, NOT fewer features.

**Where gaps show up**
- Navigation with too many flat top-level items (no grouping).
- Toolbars surfacing every action at equal weight instead of common ones + overflow.
- Onboarding asking several decisions on one screen instead of one per step.
- Forms front-loading optional fields.
- Pricing with > 3–4 tiers (analysis paralysis).
- Autocomplete/menus offering 10+ undifferentiated options.

**Detection signals**
```
→ <nav> / menu with 8+ ungrouped sibling links → S2
→ a screen presenting 6+ equal-weight primary actions with no clear primary → S2 (overchoice)
→ settings screen dumping all options flat with no sections/progressive disclosure → S2
```
Fix: group before reducing (categorization cuts apparent complexity more than removal); use smart defaults
to skip a choice; progressive disclosure for advanced options. Don't use Hick's Law to hide options users
need often.

---

## Fitts's Law — motor cost
*Time to hit a target depends on its size and distance.* `MT = a + b·log₂(2D/W)`. This governs
*operability*, so it's in scope (it's about being able to act, not how it looks).

**Practical thresholds**
- Touch targets ≥ **44×44pt** (Apple) / **48×48dp** (Material). The interactive area, not the icon — a 16px
  icon can have a 44px hit area via padding.
- Pointer targets: ≥ 24×24px, larger in dense UIs.
- High-frequency / high-consequence actions: make them larger and closer.
- Screen edges/corners are "infinite" targets — good for persistent nav.
- Destructive actions: small + distant from common ones to prevent mis-taps.

**Detection signals**
```
→ interactive element (button/icon-button/link) with computed hit area < 44×44 on touch UIs → S2/S3
   e.g. <button style={{padding:0}}><Icon size={16}/></button>  → tiny tap target
→ destructive control placed flush against a common control at equal size → S3 (mis-tap risk)
→ primary mobile action placed top-of-screen out of thumb reach → S2
```
Fix: expand hit areas with padding (not visual size); place actions near the content they affect; put
primary mobile actions in the thumb zone; keep destructive actions separated.

---

## Miller's Law — memory cost
*Working memory holds a limited number of meaningful chunks (~4±1 realistically; the famous "7±2" is
chunks, not raw items).* Cite **chunking**, not a magic number.

**Where gaps show up**
- Flat lists of 10+ nav items with no grouping.
- Long forms with no sections.
- Unformatted codes/numbers (`5558675309` vs `555-867-5309`).
- Onboarding shown as "Step 7 of 12" instead of 3–5 named phases.

**Detection signals**
```
→ long form (10+ fields) with no <fieldset>/section headings → S2
→ verification/code input not chunked → S1
→ data table / long list with no visual grouping or section headers → S1
```
Fix: structure first, count second — headings, grouping, and whitespace make chunks explicit. Adjust
density to user expertise (power users chunk more).

---

## Doherty Threshold — perceived-time cost
*Under ~400ms, users stay in flow; above it, engagement degrades.*

| Response | Perception | Required treatment |
|---|---|---|
| 0–100ms | instant | none |
| 100–400ms | fast | none / subtle |
| 400ms–1s | noticeable | loading indicator |
| 1–3s | slow | indicator, ideally progress |
| 3s+ | flow broken | progress + estimate |
| 10s+ | context switch | progress + background option |

**Where it bites:** view/slide transitions, toggles/tabs, search/filter, autocomplete (first suggestions
< 300ms), and **button press feedback** (visual state change < 100ms regardless of when the action finishes).

**Detection signals**
```
→ async interaction with no acknowledgement within ~100ms (no pressed/disabled state) → S3
→ search/filter that waits for the full response before showing anything (no skeleton) → S2
→ spinner shown for an action that completes < 400ms → S1 (the flash itself is disruptive)
```
Fix: acknowledge instantly (state change on the trigger), then loading indicator for 400ms–3s, progress
beyond 3s; use optimistic UI / skeletons to cut *perceived* wait. The threshold is about perceived wait,
not a ban on a deliberate 250ms transition.

---

## How to use these in the audit
For each interactive surface ask: too many choices at once (Hick)? hard to reach / mis-tap risk (Fitts)?
too much to remember (Miller)? does it feel laggy with no feedback (Doherty)? Each "yes" is a finding with a
behavioral fix — never a restyle.
