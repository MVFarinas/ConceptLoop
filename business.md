# Business Model

> Early thinking. Nothing here is locked; these are the choices to pick between as validation
> progresses. Tied to [target.md](target.md) (who pays) and [decisions.md](decisions.md)
> (what we built / cost structure).

## Positioning: additive tool, not a competitor

ConceptLoop is **not** a replacement for UWorld or Archer Review. It does not provide a
question bank, rationale videos, or full NCLEX content coverage. Students will likely use
it *alongside* their existing prep platform — the pitch is that UWorld/Archer give them
content and explanations; ConceptLoop fixes the part those platforms leave broken (the
50/50 miss that keeps coming back as the same question).

**Pricing context from the market:**
- Archer Review: $129/3mo (QBank + CAT) up to $400/3mo (full access)
- UWorld: ~$229/3mo (university discount codes exist)

ConceptLoop should be priced **well below** these — students are already paying $130–$400
for their primary platform. We are an additive layer, not a primary subscription. A price
point that feels like a no-brainer add-on (~$10–20/mo, or a one-time exam-cycle fee) is
more likely to convert than anything that feels like a competing commitment.

This also shapes the B2B2C pitch to UWorld/Archer: we are not a threat, we are the feature
they don't have and their students are already asking for.

---

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

## B2B2C target pipeline (priority order)

Contacts and channels identified so far — to pursue once B2C pilot has proof:

| Target | Type | Why | How to reach |
|---|---|---|---|
| **Mark Klimek / Klimek Reviews** (klimekreviews.com) | Content creator / course seller | Sells NCLEX prep courses; already has question banks + structured content. His students are our primary demographic. A partnership gives him a differentiated tool and us a vetted bank. | Direct outreach via site contact; pitch: "your students, smarter remediation." |
| **@kristine_nurseinthemaking** (Instagram) | Influencer / brand | NCLEX-focused brand; audience is exactly our target. Doesn't need to sell the product — awareness partnership or affiliate model. | Instagram DM / email via bio. Early-mover social proof is valuable before a formal B2B deal. |
| **UAlberta Faculty of Nursing** | Institution | Direct access via project owner's affiliation; built-in pilot cohort; brings vetted banks. | Internal academic channel — start here for the pilot. |
| **UWorld** | B2B2C (long shot) | The dominant NCLEX prep platform — used by project owner's cousin who recently passed. Market leader; they could build this themselves, but a partnership or acqui-hire is not impossible once proof exists. Lowest priority; highest upside. | Cold outreach only after strong traction. |
| **Archer Review** | B2B2C | Second major NCLEX platform alongside UWorld — confirmed by project owner's sister as one of the two platforms nursing students rely on most. Already markets "remediation" but delivers only explanations; our loop is exactly the substance their branding promises. Differentiated pitch: *"you already call it remediation — here's what it actually looks like."* | Cold outreach once proof exists. More reachable than UWorld. |
| **Smaller NCLEX prep companies** | B2B2C | Want differentiation from UWorld/Archer; our loop is exactly what they'd market as their edge. | Cold outreach once proof exists. |

> **IP note:** Klimek's content has reportedly been circulated via unauthorized Google Drive
> shares. Before any partnership, confirm what he considers shareable — we don't want to build
> on top of leaked content or expose ourselves to a licensing dispute. Direct partnership
> sidesteps this entirely and is the right path.

## Emerging product surface — question bank generation as a service (post-MVP idea)

A potential twist on the business model worth parking: the same underlying system
(Misconception Object + hybrid generation) could be sold as a **question bank generation
service** — not just remediating a student's misses, but generating a *fresh, varied bank*
from an existing question set + solutions. Use cases:

- **Course instructors** want exam variants that can't be shared between cohorts.
- **Content creators** (Klimek tier) want to expand their library without writing every
  question from scratch.
- **Licensing bodies** want to generate new items from a validated concept map.

This is a B2B product surface, distinct from the student-facing remediation loop, and
potentially higher willingness-to-pay per use. The creative quality of the generation is
the product — which means there's a defensible premium tier around *how different and
high-quality* the generated questions are, not just how many. Worth revisiting after the
core loop is validated. Parked in [future.md](future.md).

## Open business questions

- What's the realistic willingness-to-pay for a nursing student already paying for UWorld/Archer? (Are we additive or a replacement?)
- Can variant reuse across students with identical banks cut per-user cost enough to widen the free tier?
- Does the B2B2C motion require us to vet/host banks (liability in high-stakes fields) or stay BYO?
- Unit economics: target gross margin per paid user given LLM cost per active study session.
