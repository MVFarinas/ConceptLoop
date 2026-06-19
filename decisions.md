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
- **Not (initially) mobile-first or a plugin.** Web app first. (See Open Questions.)

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

### 4.2 Generation strategy: open-ended vs. constrained
Fully open LLM generation (flexible, higher accuracy risk) vs. constrained templates / vetted
concept bank with AI-driven variation (safer, more defensible, more work). Leaning toward
constrained variation for high-stakes domains. **Decision pending.**

### 4.3 Human-in-the-loop review
Do regenerated questions need review before being shown? Options: confidence thresholds,
student "this question was broken" flagging, or expert review for a curated set. **Pending.**

### 4.4 Re-injection scheduling
Use an existing modern SRS algorithm (FSRS) for timing, but modified so the *re-test* uses the
regenerated variant rather than the original card. **Tentative: FSRS.**

### 4.5 Form factor
Web app first (decided). Mobile / Anki plugin / API — later. Confirm web-first is acceptable
to pilot users.

---

## 5. Proposed tech stack (draft — to confirm)

> Optimized for: fast iteration, strong AI-orchestration ergonomics, and not over-building
> before validation.

| Layer | Choice | Why |
|---|---|---|
| **LLM** | **Claude (Anthropic)** — Sonnet 4.6 for the regeneration/inference loop; Opus 4.8 for harder misconception inference where quality matters most | Strong reasoning for the inference step; latest models. Pin truth via user key, so model picks the best reasoning quality/cost tradeoff per step. |
| **Frontend** | React + Next.js (TypeScript) | Web-first, fast to build, good ecosystem; SSR if/when needed. |
| **Backend** | Python + FastAPI | Best ergonomics for AI orchestration, evals, and prompt tooling. |
| **DB** | Postgres | Stores question banks, attempts, cross-off events, review queue, regenerated variants. |
| **Vector / similarity** | pgvector (in Postgres) | Concept embeddings + anti-duplication checks for regenerated questions. |
| **SRS scheduling** | FSRS (open-source algorithm) | Modern spaced-repetition timing; integrate with variant re-injection. |
| **Auth + hosting (MVP)** | Supabase (Postgres + auth) or Render/Fly + Clerk | Minimize infra work pre-validation; revisit at scale. |
| **Eval / safety harness** | Lightweight prompt-eval suite + golden set of misconception cases | Critical for high-stakes accuracy; track regeneration quality and dup-rate over time. |

**Language summary:** TypeScript on the frontend, Python on the backend. Single primary LLM
provider (Anthropic) to start.

> Stack is a recommendation, not locked. The only near-locked pieces are Postgres+pgvector and
> Claude as the LLM. Everything else is changeable before first commit of real code.

---

## 6. The minimum pipeline (reference)

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
