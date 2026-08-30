# Residuality Theory in System Design Interviews — Plan

A plan for incorporating Barry O'Reilly's **Residuality Theory** (from
`_ignore/residuality.pdf`, *Residues: Time, Change, and Uncertainty in Software
Architecture*, 2024) into this explorer, as a **Stressors** feature that mirrors
the existing `tradeoffs` mechanics.

---

## 1. What the book actually says

The core claim, in the author's own words:

> *A random simulation of stress is a better way of generating a software
> architecture than prediction, requirements analysis, reuse of patterns, or
> reactive change management by coding.*

### The concept chain

| Term | Definition (book) |
|---|---|
| **Hyperliminal system** | A complicated, ergodic, *ordered* software system executing inside a complex, non-ergodic, *disordered* business context. Software engineering is the engineering of hyperliminal systems. |
| **Naïve architecture** | The starting point: the design that solves the problem *as stated*. The "happy path architecture". Deliberately arbitrary — it exists to be stressed. |
| **Stressor** | *"Any fact about the context that is currently unknown to you."* Not a risk, not a requirement, not an edge case. No probability, no consensus, no likelihood needed — only a coherent narrative. |
| **Attractor** | A recurring phase state the *business* system falls into. A stressor pushes the business into an attractor. Attractors are far fewer than stressors — that asymmetry is the whole leverage. |
| **Residue** | What is left of the architecture after the stress. Concretely: the naïve architecture *plus the change needed to survive that attractor*. **The residue is the unit of software architecture.** |
| **Residual architecture** | All residues compressed/integrated into one coherent architecture. |
| **Hyperliminal coupling** | Invisible coupling revealed when one stressor damages two or more components. Invisible until the stressor hits. Non-functional concerns *are* these couplings. |
| **Contagion analysis** | The incidence matrix (stressors × components) that exposes hyperliminal coupling and drives refactoring. |
| **Criticality** | The right balance of N (nodes), K (links), P (bias) that survives *unknown* stressors. **The architect's goal is criticality, not correctness.** |
| **Looping** | The signal of criticality: new stressors are already survived by existing residues, with no new change needed. |

### The two-step method

1. **Stressor analysis** — randomly simulate the business environment. For each
   stressor record: *name · how it's detected · the attractor it pushes the
   business into · the business reaction · the technical change to the residue.*
   Each spreadsheet row **is** a residue.
2. **Contagion analysis** — incidence matrix, stressors as rows, components as
   columns, 1 where the stressor hits the component. Seven refactoring triggers:
   high row totals (dangerous stressors + coupling), high column totals
   (over-loaded components), >1 per row (hidden coupling), similar column
   response (merge components), many high numbers (K too high), stressor
   combinations (attack trees), zero columns (under-stressed).

### The rules (non-negotiable per the book)

- **No probability.** Ever.
- **All stressors go in the list, no matter how ridiculous.**
- **No cost/over-engineering filtering until after weaknesses are explored.**
- Purely technical stressor lists are *"not a good sign"* — stressors come from
  the **business context**: competitors, regulation, currency, war, new customer
  types.

### The empirical test

Generate a *fresh* stressor list never used in design. Count survivals for naïve
(X) and residual (Y) architectures over S stressors:

> **Ri = (Y − X) / S**, where −1 ≤ Ri ≤ 1. Ri > 0 means real movement toward criticality.

### The worked example (EV charger platform)

The canonical illustration, worth reusing as the teaching example:

- Naïve: key fob → panel → cloud API → DB.
- Stressor "key fob breaks" (Residue #3) → attractor: stranded driver, low
  battery, can't reach next charger → residue: **read the licence plate instead**;
  bill afterwards. Side effect: decouples *identity* from *charging*.
- Stressor "cars crash into chargers" (#8) → residue: spare capacity + **cameras**.
- Stressor "drivers park for hours" (#12) → residue: **sliding-scale time billing**
  (a *business* residue, not a technical one).
- **Looping payoff #1:** *ICE-ing* (petrol cars blocking chargers) — never
  designed for, but already survived by #8 + #3 + #12 combined.
- **Looping payoff #2:** the 2023 EU AFIR law mandating ad-hoc credit-card
  payment — survived 10 years later, because stressor #3 had already decoupled
  payment from identity.

**This is the single most valuable teaching artifact in the book** and should be
the flagship dataset.

---

## 2. Why this fits this repo

Residuality is a *near-perfect* fit for a system-design **interview** explorer,
for a reason the book itself supplies: senior architects already do this, they
just can't explain it. That is exactly the gap between a mid-level and a senior
interview performance.

The repo already has the right shape:

- `step.tradeoffs` — tensions a step balances → **the model to mirror.**
- `step.traps` — mistakes → per-step card list.
- `step.recap.newRisk` — already gestures at "what did this step break?"
- `step.failureDrills` — `{ scenario, expectedBehavior, mitigation }` — the
  *closest existing field*, and the one most at risk of confusion (see §6).
- `steps[].options[]` — the design already carries alternatives per step.
- `satisfies` — a Wrap-up matrix view already exists as a rendering precedent.

### The honest tension (state it, don't hide it)

The book is **explicitly hostile** to requirements, risk, patterns, and scenarios
— all of which this repo is built on (`requirements`, `patternCatalog`,
`satisfies`). Two options:

- **(a) Purist**: recast datasets around naïve → residues. Wrong call: it would
  break 90+ datasets and fight the interview format, where requirements
  *are* the stated game.
- **(b) Additive** ✅: add residuality as a **complementary lens**. The book itself
  permits this: *"Applying residuality theory to your current architectural
  process is fairly easy and doesn't demand that you abandon your current way of
  working."*

**Recommendation: (b).** But keep the concepts honest — do not silently
redefine a stressor as a risk. The distinction (no probability, business-context
origin, narrative not number) is the entire value, and the PDF's
"Stressors, Requirements, Risks, Scenarios, and Edge Cases" section should be
rendered nearly verbatim as teaching content.

---

## 3. Proposed schema

Mirrors `tradeoffs` exactly: per-step arrays, deduped into a derived
dataset-level entry, all optional, absent → renders nothing.

### 3.1 `step.stressors[]` (per-step, the primary field)

```jsonc
"stressors": [                                  // optional; what could invalidate this step's decision
  {
    "stressor":  "A regulator mandates ad-hoc card payment at every charger",
    "detection": "Trade press, legal counsel, a competitor shipping it first",
    "attractor": "Non-subscribers arrive expecting to pay at the point of use; the subscription is no longer the only path to a charge.",
    "business":  "Accept walk-up payment without abandoning the subscription base.",
    "residue":   "Payment method is decoupled from identity: the charge command carries an authorization token, not a member id.",
    "survived":  true,                          // optional; true = looping (already survived, no change needed)
    "components": ["billing", "charge-cmd"],    // optional; node ids hit — feeds the contagion matrix
    "group":     "Regulation",                  // optional; overview grouping override (PESTLE-ish)
    "icon":      "assets/icons/stressors/law.png" // optional; falls back to icons/stressor.png
  }
]
```

Field notes:
- `stressor` (**required**) — the fact outside current understanding. Narrative,
  never a probability.
- `attractor` — the *business* state, not the technical failure. This is the
  field that keeps authors honest and stops stressors decaying into "server dies".
- `residue` — the change to the architecture. The unit.
- `survived: true` — renders as a **"Already survived"** badge. This is the
  book's *looping*, the signal of criticality, and it deserves visible
  celebration in the UI.
- `components[]` — node ids from `highLevelArchitecture.nodes`, enabling the
  contagion matrix (§3.3) without a second authoring pass.

### 3.2 `stressorAnalysis` (dataset-level, optional prose framing)

```jsonc
"stressorAnalysis": {
  "naive": "Key fob → charger panel → cloud API → database.",  // the naïve architecture
  "note":  "Stressors come from the business context, not the infrastructure.",
  "residualIndex": { "naive": 3, "residual": 11, "stressors": 14 }  // optional; Ri = (Y-X)/S
}
```

`residualIndex` renders the book's empirical test with the computed **Ri = 0.57**
— a genuinely distinctive artifact no competing resource has.

### 3.3 Derived: contagion matrix (no authoring)

Built entirely from `step.stressors[].components[]` × `highLevelArchitecture.nodes`,
exactly as `stepsOverview` derives its decision tree from `steps[]`. Renders the
incidence matrix with row/column totals and highlights the book's triggers:

- row total high → dangerous stressor
- **≥2 cells in a row → hyperliminal coupling** (visually flagged — the money shot)
- column total high → over-stressed component
- column total 0 → *under-stressed*, not invulnerable

---

## 4. Rendering plan

### 4.1 Per-step section — "Stressors & Residues"

Card list appended in `renderStepExtras()`. Placement: **after** `renderStepTradeoffs`
and `renderTopConcepts`, **before** `renderTraps` — trade-offs are the decision,
stressors are what attacks it.

Card layout (reuses `.concept-card` box, new left-accent colour):

```
┌────────────────────────────────────────────┐
│ ⚡ A taxi firm wants one account, 50 drivers │  ← stressor
│ Detected: sales pipeline, inbound RFP       │  ← detection (muted, small)
│ ─────────────────────────────────────────── │
│ Attractor  Fleet buyers replace individual  │  ← the business state
│            subscribers as the growth path.  │
│ Residue    Account splits from identity;    │  ← the architectural change
│            many credentials per payer.      │
│ [billing] [auth]                            │  ← component chips
└────────────────────────────────────────────┘
```

With `survived: true`, the card gets a **✓ Already survived** badge and a muted
green accent — no `residue` line needed.

### 4.2 Overview entry — "Stressors"

Derived via `collectStepItems(data, "stressors")`, grouped by `group` (default:
step title), each card carrying step chips. Sits **next to Trade-offs** in the
Overview group. Exactly the `tradeoffs` pattern, zero new machinery.

### 4.3 Wrap-up entry — "Contagion Analysis"

The derived matrix (§3.3). Placed in `WRAPUP_ORDER` after `satisfies` — it is the
same *"does the design hold up"* family of question, viewed from stress rather
than requirements.

### 4.4 Sidebar wiring

| Slug | Group | Position |
|---|---|---|
| `stressors` | Overview | after `tradeoffs` |
| `contagion` | Wrap-up | after `satisfies` |

---

## 5. Implementation steps

Ordered, each independently verifiable.

**Phase 1 — schema + per-step rendering (the core; ships alone)**
1. `PLAN.md`: add `step.stressors[]` and `stressorAnalysis` to the schema block,
   in the existing annotated-JSONC style.
2. `_templates/interview.js`:
   - `INTRO_SLUGS`: add `stressors: "stressors"`, `contagion: "contagion"`.
   - `ICON_FALLBACK`: add `stressor: "icons/stressor.png"`, `residue: "icons/residue.png"`.
   - `makeStressorCard(item, opts)` — mirrors `makeTradeoffCard`.
   - `renderStepStressors(stressors)` — mirrors `renderStepTradeoffs`.
   - Wire into `renderStepExtras()` after concepts, before traps.
   - `validateDataset()`: add `"stressors"` to the existing
     `for (const key of ["tradeoffs", "traps"])` array-check loop; reject a
     `probability` or `likelihood` key on any stressor **with a message that
     explains why** ("residuality forbids probability — describe the attractor
     instead"). This is the schema teaching the method.
3. `_templates/styles.css`: `.concept-card.stressor-card` (accent edge),
   `.stressor-survived` badge, `.stressor-attractor` / `.stressor-residue`
   label rows.
4. Icons: add `_templates/icons/stressor.png`, `residue.png`.
5. Sample content in `data/examples/url-shortener` (canonical example dataset).

**Phase 2 — derived Overview entry**
6. `collectDatasetStressors(data)` → `collectStepItems(data, "stressors")`.
   ⚠️ **`conceptKey()` (interview.js:765) keys on `term || name || title ||
   definition || description` — it does *not* know `stressor`.** Either add
   `concept.stressor` to that fallback chain, or name the field `name` instead
   of `stressor`. Recommendation: **add `stressor` to `conceptKey`'s chain** —
   `stressor` reads far better in authored JSON than a generic `name`, and the
   one-token change is backward compatible.
7. `renderIntroStressors()` + `buildEntries()` push + `renderIntroEntry()`
   dispatch (switch on `entry.id`, interview.js:3928).

**Phase 3 — contagion matrix**
8. `buildContagionMatrix(data)` — derive from `components[]` × nodes.
9. `renderIntroContagion()` — table, totals, coupling highlight, trigger legend.
10. `WRAPUP_ORDER` placement.

**Phase 4 — content**
11. **`data/book/residuality`** — a *method* dataset, sibling to the existing
    `interview-method` and `patterns` datasets. Teaches: hyperliminality,
    stressor vs. risk/requirement/edge-case/scenario, the two-step method, the
    seven matrix triggers, the heuristics list, Ri. Uses the **EV charger**
    worked example throughout, including both looping payoffs (ICE-ing, AFIR).
12. Retrofit stressors onto 2–3 flagship cases: `payment-system`,
    `notification-system`, `flash-sale` (a natural stressor magnet).
13. `CLAUDE.md` + `AGENTS.md` (byte-identical, `cp CLAUDE.md AGENTS.md`) +
    `README.md`s — new field, new entries, new icons.

**Phase 5 — build**
14. `python3 build.py`, `node --check`, JSON validation, HTTP-serve check, commit `docs/`.

---

## 6. Risks and decisions to make

1. **Overlap with `failureDrills`.** `{ scenario, expectedBehavior, mitigation }`
   is structurally close to a stressor. **They are not the same and the docs must
   say so**: a failure drill is a *technical* failure of a known component
   (linear, chaos-engineering-flavoured); a stressor is a *business-context* fact
   outside current understanding. The book explicitly separates these ("Chaos
   engineering is concrete and linear, and unrelated to the lateral concept of
   stressors"). Risk: authors will fill `stressors` with "the database dies".
   **Mitigation**: the `attractor` field is required-in-spirit — you cannot write
   a business attractor for "the DB dies" without noticing you're in the wrong
   field. Consider a validator warning when `stressor` matches technical-failure
   patterns.

2. **Authoring cost across 90+ datasets.** Don't retrofit everything. The field
   is optional; 3–5 exemplars plus the method dataset is a complete, shippable
   feature. Trying to backfill all datasets would be weeks of low-value content.

3. **`components[]` accuracy.** The contagion matrix is only as good as the node
   ids. Mitigation: validate ids against `highLevelArchitecture.nodes` (same
   check style as `view.highlight`), and treat the matrix as optional — it
   renders only when ≥1 stressor declares components.

4. **Dogma calibration.** The book is polemical ("these traditional ideas are
   harmful"). This repo teaches interviews, where requirements analysis is the
   stated game. Present residuality as *the senior lens that survives the
   follow-up question*, not as a replacement for requirements. Attribute clearly
   to O'Reilly and link the source.

5. **Interview realism.** In a 45-minute interview nobody runs 200 stressors.
   The pitch should be: **3–4 well-chosen stressors, one looping payoff** is what
   separates a senior signal from a mid one. Frame the feature around that
   budget, and consider a `levelVariants`-style note on how deep to go.

---

## 7. Why this is worth doing

The differentiator is **looping**. Every system-design resource on the internet
lists trade-offs. Almost none can show a design *surviving a stressor it was
never designed for* — and that is the exact moment an interviewer decides
somebody is senior. The EV-charger AFIR story (a 2013 design absorbing a 2023
law for free) is the most compelling single artifact in the book, and this repo
already has the rendering machinery to tell it.

---

## Appendix: source

`_ignore/residuality.pdf` — Barry M. O'Reilly, *Residues: Time, Change, and
Uncertainty in Software Architecture*, Leanpub, 2024-06-02.
Underlying papers: O'Reilly, B. M. (2022). "Residuality Theory, random
simulation, and attractor networks." *Procedia Computer Science*, 201, 639–645.
