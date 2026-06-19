# Decisions — Scope, Architecture & Open Questions

This is the authoritative scope/tech document. When something here conflicts with
[initial_inspiration.md](initial_inspiration.md), this file wins.

---

## 1. What ConceptLoop IS

- A layer that sits **on top of a question bank the user already has.**
- A system that captures **how** a student answers (especially the cross-off / elimination
  process), not just **what** they answered.
- A misconception-inference + question-regeneration engine: miss → infer the specific gap →
  generate a structurally different question targeting that gap → re-inject into a
  spaced-review queue.
- A twist on Anki/SRS that is genuinely more involved: variation of practice, not exact repetition.

## 2. What ConceptLoop is NOT (out of scope)

- **Not a test-bank generator from notes.** Commoditized; many tools already do it; it dilutes
  the defensible core. Users bring their own questions.
- **Not a content library.** We don't author or sell question content.
- **Not a source of truth for correctness.** The LLM never decides what the right answer is.
- **Not (initially) mobile-first or a plugin.** Web app first → companion mobile app post-MVP.

---

## 3. Locked design decisions

### 3.1 Users supply the questions AND the verified answer key
This is the keystone safety decision. In high-stakes domains (nursing, medicine, law) we
**cannot** have the LLM guessing the correct answer. The user (or their source material)
provides ground truth. The LLM reasons *over* that ground truth; it never produces it.

### 3.2 The LLM's role is "reasoning engine over ground truth," not "knowledge oracle"
Given (stem + options + crossed-off set + chosen answer + **verified correct answer**), the
LLM's jobs are:
1. **Contextualize** — explain how the verified correct option is confirmed by the answer key,
   and how it differs from the option(s) the student was stuck on.
2. **Infer the misconception** — why was this student plausibly stuck between *these specific*
   options, assuming the key is correct?
3. **Regenerate** — produce a new question on the same concept from a different angle/context.

Because truth is pinned by the user-supplied key, hallucination risk drops from "invents
medical facts" to "writes a suboptimal rephrase" — a much more tolerable failure mode.

### 3.3 The cross-off interaction is a first-class feature
Eliminating options is encouraged and captured. It is the signal that distinguishes a
*discrimination* failure from a *deeper gap* (see [scenario.md](scenario.md)).

### 3.4 Anti-duplication is an explicit requirement
Regenerated questions must differ in **angle/context/framing**, not just wording. We will
guard against near-duplicate rewordings (the known LLM failure mode) — likely via embedding
the concept and the generated variant and rejecting variants too similar to the original.

### 3.5 Narrow first wedge
One domain + one format for the MVP: **NCLEX-style nursing content, single-best-answer MCQs.**
See [target.md](target.md).

### 3.6 The only external input is the imported bank + answer key
The **single** thing that comes from outside ConceptLoop is the user-imported MCQ bank and its
verified solutions. Everything after import — the attempt experience, cross-off capture,
misconception inference, regeneration, the review queue — is **fully in-house on our web/app.**
We are not a thin wrapper around someone else's platform; the entire interactive experience is ours.

### 3.7 Privacy & data isolation (current requirement, not future)
A user's account data and uploaded banks are **private to that user by default.** No student
can see, pull from, or be served questions out of another user's bank. This is a hard
requirement from day one — every storage and retrieval path is scoped per-user/tenant.
The **only** future exception is an opt-in community sharing feature (post-MVP) — see
[future.md](future.md), which also covers the intellectual-property questions that opens up.

### 3.8 MVP scope and philosophy — locked

**Target timeline:** 1–2 weeks to a working, pilotable product.

**Philosophy:** validate fast, not polish first. The goal is to get a real user (starting
with one nursing student) through the full loop and find out if the core experience —
miss a question, get a meaningfully different one back later — actually lands. Every
feature that doesn't serve that validation is deferred.

**In scope for MVP (priority order):**

1. **Exam UI with cross-off interaction** — the keystone feature. Student can eliminate
   options, navigate between questions, and mark a final answer.
2. **Question navigation** — back/forward between questions within a session; efficient,
   not just linear.
3. **Submission + grading** — submit the session, see a score.
4. **Missed question follow-up page** — for each incorrect answer: the misconception
   inference output, plain-language explanation, and the reworded/recontextualized variant
   question. This is the product. It must feel different and meaningful, not like a tooltip.
5. **Front, back, and database built for expansion** — even if the feature set is minimal,
   the data model and architecture must cleanly support the full pipeline (caching, ledger,
   FSRS re-injection) without a rewrite.

**Explicitly deferred (post-MVP):**
- Concept dashboard / analytics / misconception report (the ledger analytics surface)
- FSRS-based automatic re-injection into a spaced review queue (manual "review missed"
  is fine for the pilot)
- Temporal decay tracking
- User account settings, onboarding polish
- Mobile / responsive optimization beyond basic usability

**The north star for MVP quality:** it doesn't need to be polished. It needs the AI
integration to feel *meaningful* — the reworded question should clearly feel like it's
targeting what the student got wrong, not like a random rephrasing. Graceful execution
of the core AI features matters more than any surrounding UI chrome.

### 3.9 Risk stance — ship punchy, dial back on evidence
This is a novel concept. The default is to lean into creative, risk-tolerant choices now
and tighten only where real usage shows a failure. We do not over-engineer safety rails
before we've seen what actually breaks. User flagging + auto-fallback (§4.2) is the
mechanism that makes this safe to do — problems surface and are addressed without manual
intervention.

---

## 4. Open questions (need a decision before/early in build)

### 4.1 The "wrong answer, no cross-off used" case  ⚠️ priority
Confident students sometimes don't use cross-off (they have the answer in mind and just pick
the match). If they're **wrong** and gave us no elimination signal, what happens?

Candidate paths (to decide):
- **(a) Fall back to explanation** (what everyone else does) — safe but undifferentiated.
- **(b) Prompt a lightweight retry**: "Before we explain — which one were you torn between?"
  to recover the signal after the fact.
- **(c) Infer from the wrong answer alone**, with lower confidence, and still regenerate.
- Likely a blend: try (b), fall back to (a)/(c). **Decision pending.**

### 4.2 Generation strategy — Locked: hybrid with user flagging + auto-fallback
Open generation anchored to the structured Misconception Object (not raw text generation),
combined with:
- **User flagging** ("this question felt broken / repeated itself") as the primary quality
  signal. High flag rates on a concept → tighten constraints or fall back to the constrained
  model for that domain. Flag data also feeds prompt tuning over time.
- **Lightweight model self-check** before surfacing the question: "does this have a single
  defensible best answer given the key?" Low-confidence outputs are held back.
- **Auto-fallback rule:** if flag rate on a concept or domain exceeds a threshold, the system
  automatically switches to a constrained template approach for that domain. No manual
  intervention needed to dial down quality risk.

This is intentionally punchy: start with creative open generation, let real usage tell us
where it breaks, then tighten only where it actually fails. See §3.8 (risk stance).

### 4.3 Human-in-the-loop review
Do regenerated questions need review before being shown? Options: confidence thresholds,
student "this question was broken" flagging, or expert review for a curated set. **Pending.**

### 4.4 Re-injection scheduling
**Locked: FSRS.** Open-source modern spaced-repetition algorithm (Anki's current default).
Models per-card *stability* (how long you'll retain it) and *difficulty* (how hard it is for
you) to predict the optimal review date. FSRS handles the *when*; ConceptLoop handles the
*what* (regenerated variant instead of original card). They slot together cleanly.

### 4.5 Form factor
**Locked: web app first.** Companion mobile app is a post-MVP feature — see [future.md](future.md).

---

## 5. Proposed tech stack (draft — to confirm)

> Optimized for: fast iteration, strong AI-orchestration ergonomics, and not over-building
> before validation.

| Layer | Choice | Why |
|---|---|---|
| **LLM** | **Claude (Anthropic)** — Sonnet 4.6 for the regeneration/inference loop; Opus 4.8 for harder misconception inference where quality matters most | Strong reasoning for the inference step; latest models. Pin truth via user key, so model picks the best reasoning quality/cost tradeoff per step. |
| **Frontend** | React + Next.js (TypeScript) | Web-first, fast to build, good ecosystem; SSR if/when needed. |
| **Backend** | Python + FastAPI | Best ergonomics for AI orchestration, evals, and prompt tooling. |
| **DB** | Postgres | Stores question banks, attempts, cross-off events, review queue, regenerated variants, misconception ledger. |
| **Caching** | Postgres `DistractorSemantics` table (persistent) + `cachetools` (in-process memory) | No extra service needed. Check in-process cache → DB row → LLM call, in that order. Only fires on a miss. `cachetools` is a tiny Python library, no Redis, no new infra. |
| **Vector / similarity** | pgvector (in Postgres) | Concept embeddings + anti-duplication checks for regenerated questions. |
| **SRS scheduling** | FSRS (open-source algorithm) | Modern spaced-repetition timing; integrate with variant re-injection. |
| **Auth + hosting (MVP)** | Supabase (Postgres + auth) or Render/Fly + Clerk | Minimize infra work pre-validation; revisit at scale. |
| **Eval / safety harness** | Lightweight prompt-eval suite + golden set of misconception cases | Critical for high-stakes accuracy; track regeneration quality, dup-rate, and flag rate over time. |

**Language summary:** TypeScript on the frontend, Python on the backend. Single primary LLM
provider (Anthropic) to start.

> Stack is a recommendation, not locked. The only near-locked pieces are Postgres+pgvector and
> Claude as the LLM. Everything else is changeable before first commit of real code.

---

## 6. The minimum pipeline (reference)

> The misconception-inference half of this pipeline is designed in depth in
> [inference.md](inference.md) — the core IP.

```
user-supplied question bank + verified answer key
        │
        ▼
student attempts MCQ ──► captures: crossed-off options, stuck-between pair, chosen answer
        │
        ▼  (if wrong)
LLM step 1: contextualize chosen vs. correct vs. stuck-between, against the key
        │
        ▼
LLM step 2: infer the specific misconception (discrimination failure vs. deeper gap)
        │
        ▼
LLM step 3: regenerate a structurally different question on the same concept
        │
        ▼
anti-duplication check (embedding similarity vs. original) ──► reject & retry if too close
        │
        ▼
re-inject variant into FSRS-scheduled review queue
```
