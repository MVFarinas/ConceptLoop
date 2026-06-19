# Misconception Inference Pipeline (Core IP)

This is the deepest part of ConceptLoop and the most defensible. It turns a missed MCQ —
plus the student's cross-off behavior — into a **specific, structured, actionable
misconception** that downstream regeneration can target. It is the upstream fix for the
"near-duplicate reworded question" failure mode: if the misconception is precisely named,
regeneration is anchored to a *concept relationship*, which naturally produces a different
angle rather than a reworded sentence.

> High-level scope lives in [decisions.md](decisions.md); this file is the detailed design of
> the inference half of the loop. Regeneration consumes the **Misconception Object** defined here.

---

## 0. Guiding principles

1. **Reason over ground truth; never decide truth.** The user supplies the verified answer
   key. Every inference is "given this is correct, why was the student stuck?" — never "what
   is correct?" (See [decisions.md §3.2](decisions.md).)
2. **Cross-off data bounds the problem.** We don't ask "why did they miss this?" We ask "given
   they confidently eliminated A and E, chose B, but the answer is D — what's the confusion
   between B and D?" A well-posed, narrow question.
3. **Distractors carry latent misconceptions.** Well-constructed banks design each wrong option
   to catch a specific error. Pre-analyzing distractors against the key gives us a strong prior
   before the student acts.
4. **Structure beats free text.** A misconception must be a structured object, not a blob — so
   we can aggregate across questions, detect patterns, drive regeneration, and *measure* whether
   remediation worked.
5. **Allowed to say "I don't know."** Confidence gating is mandatory. If signal is weak, fall
   back to a plain explanation and re-test — do **not** fabricate a misconception.
6. **One miss is a hypothesis; a pattern is a diagnosis.** Confidence rises as the same
   concept-confusion recurs across items. We maintain a per-student misconception ledger.

---

## 1. Inputs available at miss time

Per attempt:
- Question stem + all options (text)
- **Verified correct answer** (user-supplied ground truth)
- Student's chosen answer
- **Crossed-off set** (options confidently eliminated)
- **Stuck-between set** (options not crossed off — includes the chosen one)
- Behavioral signals: time on question, elimination order (if captured), optional self-rated confidence

Pre-computed per question (cached — see §2):
- Concept tag(s) for stem + correct answer
- **Distractor semantics map** — what misconception each wrong option is designed to catch

Per student (accumulated — see §6):
- Prior attempts/misses on the same concept; current misconception ledger

---

## 2. Stage 0 — Question enrichment (once per question, cached, offline)

Run **before** any student sees the question, over the answer key (safe, and reusable across
every student who ever attempts it — and across community-shared banks later).

For each question, the LLM produces:
- **Concept tag(s)** — the concept the correct answer tests, mapped to a per-domain concept map.
- **Distractor semantics** — for each wrong option: *what would choosing this reveal?*
  e.g. *"Option B (calcium): chosen by students who know an electrolyte affects cardiac
  conduction but confuse calcium's role with potassium's."*
- **Concept embedding** — for aggregation and anti-duplication downstream.

Why cache: amortizes LLM cost across all attempts of that question, and turns the expensive
"name the misconception" work into a fast lookup at miss time. This is also where domain
experts (or the bank itself) can review/correct distractor semantics for high-stakes accuracy.

---

## 3. Stage 1 — Signal classification (deterministic, ~no LLM)

Pure logic on the attempt data — cheap, fast, no model call. Outputs `miss_type`:

| miss_type | Condition | Meaning |
|---|---|---|
| `discrimination_failure` | Correct answer was **not** crossed off; chosen ≠ correct; small stuck-between set | Understands the area, can't separate two close concepts. (scenario.md case 1) |
| `deeper_gap` | Correct answer **was crossed off** | The right concept wasn't even in consideration. More serious. (scenario.md case 2) |
| `no_signal` | No cross-off used at all | The §4.1 fork — weak signal. |
| `likely_slip` | Large/near-random stuck-between set, or very fast time, inconsistent with prior mastery | Possibly careless, not conceptual. |

`miss_type` routes everything downstream. Don't spend a model call to compute what a rule can.

---

## 4. Stage 2 — Misconception inference (the core LLM call)

The defensible step. Use the **higher-quality model here** (Opus-class) — this is where quality
is the product.

**Inputs:** stem, options, correct answer, chosen, crossed-off, stuck-between, `miss_type`,
pre-computed distractor semantics + concept tags, and the student's prior history on this concept.

**Output — the Misconception Object (structured):**
```json
{
  "concept": "cardiac-conduction-electrolytes",
  "type": "confusion | missing_property | stem_misread | overgeneralization | slip",
  "statement": {
    "confuses": ["potassium-role", "calcium-role"],   // for type=confusion
    "missing_property": null,
    "detail": "Recognizes electrolytes affect conduction but attributes the arrhythmia-driving role to calcium instead of potassium."
  },
  "plain_language": "You knew it was an electrolyte and cardiac-related, but mixed up which one most directly drives arrhythmias.",
  "confidence": 0.0_to_1.0,
  "evidence": ["did not cross off correct (K+)", "chose Ca2+, the engineered near-neighbor distractor", "prior miss on same concept 4 days ago"],
  "recommended_remediation": "discriminate_two_concepts | reteach_then_test | reinforce | none"
}
```

The structured `type` + `statement` are what make aggregation, regeneration, and evaluation
possible. The distractor-semantics prior means the model is usually *identifying which known
trap* was hit, not inventing one — which is both more accurate and more auditable.

Routing nuance by `miss_type`:
- `discrimination_failure` → focus the inference on the boundary between the surviving correct
  answer and the chosen near-neighbor.
- `deeper_gap` → the model should explain what concept was wrongly excluded; remediation leans
  `reteach_then_test`.
- `no_signal` → see §5; lower confidence ceiling, or recover signal first.
- `likely_slip` → bias toward `type: slip`, `remediation: reinforce/none`.

---

## 5. Stage 3 — Confidence gate & routing

The Misconception Object's `confidence` decides the action — this is the anti-hallucination valve:

| Confidence / case | Action |
|---|---|
| High + specific | Proceed to **targeted regeneration** against the named misconception. |
| `no_signal` (no cross-off, wrong) | Resolve via the [§4.1 fork](decisions.md): try to recover signal ("which were you torn between?"), else fall back to plain explanation + re-test original. **Do not fabricate a misconception.** |
| Low confidence | Plain explanation now; re-queue the **original** question; gather more signal before committing to a misconception. |
| `likely_slip` | Light touch: brief explanation, re-queue original later. No regeneration. |

The product is allowed — and expected — to *not* guess. A wrong remediation in nursing is worse
than a generic-but-correct explanation.

---

## 6. Stage 4 — The student misconception ledger (aggregation + temporal decay)

A single miss yields a *suspected* misconception. The ledger upgrades suspicion to diagnosis
and tracks it over time:

- Per student: `concept → misconception → {status, confidence, history, last_seen_at, decay_rate}`
- `status`: `suspected → confirmed → resolving → resolved → decayed`
- A second miss on the same concept-confusion flips `suspected → confirmed`.
- Confirmed misconceptions get prioritized for regeneration; one-off slips don't over-trigger.

**Temporal decay layer (on top of FSRS):**
FSRS handles question-level stability decay ("when should I re-see this question?"). The
ledger adds misconception-level decay: a `resolved` misconception slowly loses confidence
if the concept hasn't been touched in a long time. After a configurable dormancy window
(e.g. several months of no activity on that concept), status flags as `decayed` and the
system surfaces a low-friction check-in — not a full re-teach, but a pulse check. This
models the real experience of forgetting (Python syntax, electrolyte interactions, legal
doctrine) after extended time away from a subject.

This is distinct from FSRS's job. FSRS: *when to re-show a question.* Ledger decay: *when
to re-validate that a previously resolved misconception hasn't silently resurfaced.*

**Account integrity note:**
The ledger is **strictly per-user** and is only useful because it is. Sharing an account
corrupts both users' misconception maps — the model degrades silently and neither person
gets accurate remediation. ConceptLoop's ToS must prohibit account sharing, and onboarding
should make the personal nature of the model explicit: *"Your misconception map is yours
— the more you use it as an individual, the better it works."*

This ledger is also the **paid analytics surface** ("your misconception report" — see
[business.md](business.md)) and the thing that makes ConceptLoop feel like it *understands the
student*, not just scores them.

---

## 7. Stage 5 — Closing the loop (did remediation work?)

When the regenerated variant is later answered, feed the result back:
- **Correct** → evidence the misconception is resolving → decay confidence, move toward `resolved`.
- **Wrong again, same pattern** → misconception is *confirmed and stubborn* → escalate: switch
  to `reteach_then_test`, or flag for human/expert review.

This is what converts one-shot guessing into a longitudinal model — and gives us a hard
**success metric**: did targeting the misconception resolve it faster than re-showing the
original would have? That metric is also the core validation of the entire product hypothesis.

---

## 8. How this beats the near-duplicate failure mode

The known LLM trap is producing "what year was X" vs "in what year was X." We avoid it
*upstream*: regeneration isn't told "reword this question" — it's told "write a question that
forces the student to distinguish **potassium's vs. calcium's role in cardiac conduction**, in a
new clinical context." Anchoring to a concept *relationship* (not a sentence) yields a genuinely
different angle by construction. The embedding from Stage 0 is a final guardrail: reject any
variant too similar to the original.

---

## 9. Evaluation harness (build alongside, not after)

Because this is high-stakes, inference quality must be measured, not assumed:
- **Golden set**: a labeled set of `attempt → expected misconception`, validated by nursing
  experts / pilot users, per concept.
- Track: misconception-inference accuracy, confidence calibration, false-misconception rate,
  variant duplication rate, and loop-closure success rate (§7).
- Re-run on every prompt/model change to catch regressions.

This harness is also our credibility story for a B2B2C pitch to nursing programs.

---

## 10. Data model sketch (inference-relevant)

```
Question(id, stem, source_bank, ...)
Option(id, question_id, text, is_correct)              # is_correct from user-supplied key
ConceptTag(id, label, domain, embedding)
DistractorSemantics(option_id, reveals_misconception, concept_tag_id)   # Stage 0, cached
Attempt(id, student_id, question_id, chosen_option_id,
        crossed_off[], stuck_between[], time_ms, self_confidence)
MisconceptionInference(id, attempt_id, concept_tag_id, type, statement_json,
                       plain_language, confidence, evidence_json, remediation)
StudentMisconception(student_id, concept_tag_id, misconception_key,
                     status, confidence, history_json)                  # the ledger
RegeneratedVariant(id, targets_misconception_id, source_question_id,
                   stem, options_json, embedding)
```

---

## 11. Open decisions for this pipeline

1. **Misconception representation** — fully open structured text vs. a **predefined per-domain
   misconception taxonomy**. *Recommendation:* hybrid — open generation constrained to the
   schema in §4, with concept tags drawn from a per-domain concept map. Taxonomy improves
   aggregation/eval; pure-open is more flexible but harder to measure. **Pending.**
2. **Distractor pre-analysis timing** — **Locked: on-demand, with per-question caching.**
   Analysis fires only when a miss actually occurs (no wasted API calls on correct answers).
   Result is cached after the first miss so repeated misses on the same question don't re-run it.
3. **Confirmation threshold** — how many corroborating misses flip `suspected → confirmed`?
   *Recommendation:* 2 on the same concept-confusion, tunable. **Pending — team discussion.**
4. **§4.1 "no cross-off" recovery** — prompt-to-recover vs. infer-from-chosen-alone vs. plain
   fallback. Cross-referenced from [decisions.md §4.1](decisions.md); resolve with pilot data.
