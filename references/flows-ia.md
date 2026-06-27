# Reference: Flows & Information Architecture

Where the previous lenses look at single moments, this one looks at the **journey**: can users get where
they're going, understand where they are, back out safely, and never lose progress? In a codebase these
gaps live in routing, multi-step components, navigation structure, and the wiring between screens.

---

## User-flow analysis (walk the paths)

For each key task (sign-up, checkout, create-X, invite, settings change), reconstruct the flow from the
code and check it as a graph of screens, decisions, and branches.

Map: **entry point → happy path → decision/branch points → error paths → exit/completion.** The happy path
is the easy 20%; the UX gaps are in the missing 80%.

**What to look for**
```
→ Dead end: a screen with no forward action and no way back (route renders, no navigation out) → S3
→ No exit from a committed flow (checkout/onboarding only leaves by completing) → S4
→ Missing error branch: a decision point (payment fails, validation fails) the code never renders a
   state for → user hits a blank/crash → S3
→ Lost progress: navigating away from a long flow loses entered data with no draft/save → S3
→ Orphaned screen: a route reachable by URL but linked from nowhere, or vice-versa → S2
→ Async/background step with no status surfaced to the user (webhook, processing) → S2
```
For each flow, annotate: screen names, decision criteria at each branch, error conditions, system/async
events, and any time delays. Offer to render a flow diagram (entry oval → screen rectangles → decision
diamonds → end) if it helps the team.

---

## Information architecture (can they find it?)

IA is how content/features are categorized, labeled, and connected. Poor IA means users can't find things
even when every screen is individually fine.

**IA heuristics**
- **Findability** — can a user reach any item in ≤ 3 clicks from any entry point?
- **Discoverability** — do users encounter relevant things they weren't explicitly seeking?
- **Wayfinding** — do users always know where they are, how they got there, how to get back?
- **Scent** — do nav labels/categories accurately predict what's inside? (weak scent = wrong clicks)
- **Depth vs breadth** — prefer ≤ 3 levels for primary content; very deep hierarchies hurt.

**Detection signals (from routes & nav components)**
```
→ deep route nesting (4+ levels) for primary content with no breadcrumbs → S2 (wayfinding)
→ no breadcrumb / "you are here" indicator on a deep hierarchy → S2
→ nav labels that are internal jargon, not user vocabulary → S2 (weak scent)
→ the same destination reachable by inconsistent labels in different menus → S2
→ no global way back to a "home"/root from deep pages → S2
→ active nav item not marked as current (aria-current/visual state) → S2 (wayfinding; also recognition)
```
Navigation model to verify exists and is consistent: **global** (everywhere), **local** (section-level),
**utility** (account/settings/help), **contextual** (inline links between related content).

---

## Onboarding (first-run experience)

Goals, in priority order: get to value fast → orient (don't educate everything) → build confidence with
early wins → reduce setup friction (collect only what's needed now).

**Patterns**
- Progressive onboarding: teach features in context, at the moment they're relevant (tooltips on first
  use, empty-state prompts). Best for complex tools.
- Setup wizard: linear required configuration. Keep steps minimal; show progress; allow skipping optional
  steps.

**Detection signals**
```
→ complex product with no first-run guidance and a blank initial dashboard → S2 (also H10d)
→ setup wizard with no skip on optional steps → S2 (forces friction, loses users)
→ onboarding that front-loads many decisions before any value → S2 (Hick + time-to-value)
→ empty states that don't double as onboarding ("No projects — create your first") → S2
```

---

## Search UX (when the product has search)

- Input: placeholder describes what's searchable ("Search products, brands…"), auto-focus on open, submit
  on Enter, visible clear/reset once a query exists.
- Suggestions: after 2–3 chars; recent → trending → predicted; highlight the typed portion; cap at 5–8
  (Hick); keyboard-navigable.
- Zero results: distinct, helpful state with a path forward (clear filters, broaden query, suggestions) —
  never a bare "No results".
- Search-as-navigation: many users search to navigate ("settings", "invoices") — support it.

**Detection signals**
```
→ search with no zero-results state (or one reusing generic empty copy) → S2
→ no clear/reset control once a query is entered → S1
→ no recent/suggested queries on a frequently-used search → S1
→ filters with no visible "active filter" indication or count → S2 (recognition)
```

---

## How to use this lens
Pick the 3–5 flows that matter most to the product's value, trace each in code end to end including the
unhappy branches, then step back to the IA: could a user *find* the entry to each of those flows in the
first place? Findings here are usually the highest-severity because they block whole tasks.
