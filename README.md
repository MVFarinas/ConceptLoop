# ConceptLoop

AI-powered adaptive question reinforcement for studying. When a student misses a
multiple-choice question, ConceptLoop captures *how* they answered (especially which options
they crossed off), infers the **specific misconception** behind the miss, and later
re-presents the same concept as a **structurally different question** — testing understanding,
not memorized wording. A more involved twist on Anki/spaced repetition.

## Documents

| File | What it covers |
|---|---|
| [roadmap.md](roadmap.md) | The chosen direction — research-led, product-bound — and the phased plan to hit ML/AI, community, and startup markers. |
| [research.md](research.md) | The research/benchmark track (synthetic student + leaderboard + judge) — the public ML/AI artifact, built first. |
| [scenario.md](scenario.md) | The universal "50/50 gut-feeling" moment the product is built around (also the marketing narrative). |
| [decisions.md](decisions.md) | Authoritative scope, locked decisions, tech stack, the reference pipeline. |
| [inference.md](inference.md) | The misconception-inference pipeline — the core IP. |
| [business.md](business.md) | Business model options (B2C → B2B2C → API). |
| [target.md](target.md) | Target demographic (NCLEX nursing wedge) and pilot plan. |
| [future.md](future.md) | Post-MVP parking lot (cross-platform, community sharing, IP). |
| [initial_inspiration.md](initial_inspiration.md) | Original handoff doc — background, not spec. |

---

## ⬜ Open decisions — to discuss as a team

Decisions that are deliberately left open. Each needs a team conversation before it's locked.

### Misconception inference pipeline (from [inference.md §11](inference.md))
- [ ] **Misconception representation** — fully open structured text vs. a predefined per-domain
  misconception taxonomy. *(Lean: hybrid — open generation constrained to a fixed schema, with
  concept tags from a domain concept map.)* **Team discussion needed.**
- [x] **Distractor pre-analysis timing** — **on-demand, with per-question caching.** Fires only on a miss; result cached so repeat misses on same question are free.
- [ ] **Confirmation threshold** — how many corroborating misses flip a misconception from
  "suspected" to "confirmed"? *(Lean: 2, tunable.)* **Team discussion needed.**

### Scope & product (from [decisions.md §4](decisions.md))
- [ ] **The "wrong answer, no cross-off used" case** ⚠️ — fall back to a plain explanation,
  prompt the student to recover the signal, or infer from the wrong answer alone? (Resolve with
  pilot data.)
- [x] **Generation strategy** — **hybrid with user flagging + auto-fallback.** Open generation anchored to the Misconception Object; user flags drive tuning; high flag rates trigger auto-fallback to constrained mode per domain.
- [ ] **Human-in-the-loop review** — formal expert review layer, or rely on user flagging + auto-fallback? (May resolve with pilot data.)
- [x] **Re-injection scheduling** — **FSRS.** Handles the *when*; ConceptLoop handles the *what*.
- [x] **Form factor** — **Web-first.** Companion mobile app is post-MVP.

### Tech stack (from [decisions.md §5](decisions.md))
- [ ] **Confirm the proposed stack** — Python/FastAPI backend, Next.js/TypeScript frontend,
  Postgres + pgvector, FSRS, Claude as the LLM. (Only Postgres+pgvector and Claude are near-locked.)

### Business (from [business.md](business.md))
- [x] **Business model sequence** — **B2C first** (pilot with real students), then B2B2C. Likely B2B2C targets in priority order: nursing programs, smaller NCLEX prep companies, professional licensing bodies.
- [x] **Positioning** — **additive tool, not a competitor to UWorld/Archer.** Price point well below their tiers (~$10–20/mo or per exam cycle). Students pay $129–$400 for their primary platform; we're the no-brainer add-on.
- [ ] **Primary B2B2C partner to pitch** — which nursing school / prep company to approach first? Decide once B2C pilot has proof.
- [ ] **Exact pricing** — needs validation with pilot users on willingness-to-pay.

### Post-MVP (from [future.md](future.md))
- [ ] **Community bank sharing** — opt-in only; IP/copyright framework must be resolved first.
- [ ] **Intellectual property** — ownership framework for shared & AI-regenerated content.
