# Business Model

> Early thinking. Nothing here is locked; these are the choices to pick between as validation
> progresses. Tied to [target.md](target.md) (who pays) and [decisions.md](decisions.md)
> (what we built / cost structure).

## The defining cost reality

Every miss can trigger 2–3 LLM calls (contextualize → infer → regenerate, plus possible
anti-dup retries). **Compute cost scales with usage**, unlike a static flashcard app. This
shapes the entire model: we can't offer unlimited regeneration on a cheap flat plan without
controlling cost. Three levers: model tiering (cheap model for easy steps, premium only where
it matters), caching/reuse of variants across users with the same bank, and usage-aware pricing.

## Model options (pick a primary, keep one as fallback)

### Option A — B2C freemium subscription (recommended starting point)
- **Free tier:** bring your own bank, capped number of AI regenerations/month (e.g. enough to
  feel the magic, not enough to lean on it). Cross-off + basic SRS unlimited.
- **Paid tier (~$8–15/mo):** unlimited (or high-cap) regeneration, deeper misconception
  reports, progress analytics.
- **Why:** matches the prosumer reality that users already bring their own content; fastest
  path to revenue; aligns price with the AI cost driver.
- **Risk:** B2C churn after exams; CAC; need a strong free-to-paid hook.

### Option B — B2B2C: sell to nursing schools / test-prep companies
- Schools or prep providers license ConceptLoop for their cohorts; they supply the (already
  vetted) question banks and answer keys — which conveniently solves our "users provide ground
  truth" requirement at scale and with higher quality.
- **Why:** higher contract value, lower churn, banks come pre-vetted (huge accuracy win),
  built-in distribution to motivated students.
- **Risk:** long sales cycles; institutional procurement is slow; needs proof first.

### Option C — Add-on / API layer for existing platforms
- License the "miss → regenerate → re-inject" pipeline to platforms that already have banks
  (the UWorld/Archer tier) as the remediation engine they don't have.
- **Why:** they have content + distribution; we have the defensible loop.
- **Risk:** they may build it themselves; we become a feature, not a product; weak moat.

## Recommended sequence

1. **Validate with B2C (Option A)** on the narrow NCLEX wedge — cheapest way to prove students
   value question *regeneration* over explanation, and to gather the misconception-inference
   golden set.
2. **Convert proof into B2B2C (Option B)** — nursing programs are the natural buyer and bring
   vetted banks, which de-risks accuracy. This is likely the bigger, stickier business.
3. **Option C only opportunistically** — as a partnership, not the core bet (weakest moat).

## Pricing principles

- Tie what's metered to **AI regenerations**, since that's the true cost driver.
- Don't meter the cross-off/SRS basics — those are cheap and they're the habit-forming hook.
- Keep a generous-enough free tier that a student feels the "the 50/50 came back as a real
  question and I finally got it" moment at least a few times before paying.

## Open business questions

- What's the realistic willingness-to-pay for a nursing student already paying for UWorld/Archer? (Are we additive or a replacement?)
- Can variant reuse across students with identical banks (e.g. a shared NCLEX bank) cut per-user cost enough to widen the free tier?
- Does the B2B2C motion require us to vet/host banks (liability in high-stakes fields) or stay BYO?
- Unit economics: target gross margin per paid user given LLM cost per active study session.
