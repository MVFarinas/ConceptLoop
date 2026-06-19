# Future / Post-MVP Parking Lot

Things we explicitly want **later**, deliberately kept out of the MVP so scope stays tight.
Nothing here should pull focus from the core loop (see [decisions.md](decisions.md)) until
that loop is validated.

---

## 1. Cross-platform: web + mobile with session continuity (QoL)

- ConceptLoop available on **both web and a mobile app**, with a shared account.
- **Resume-where-you-left-off:** a student can start an exam/review session on one device and
  continue on another on the fly. Built-in tracking of session position, queue state, and
  partial attempts syncs across devices.
- Framed as a quality-of-life feature — it makes the product feel native to how students
  actually study (laptop at the desk, phone on the bus), but it is **not** part of proving the
  core hypothesis, so it waits until after MVP validation.
- *Architecture note for whoever picks this up:* designing the data model so session/queue
  state is server-authoritative from the start (even in the web-only MVP) makes this far
  cheaper to add later. Worth keeping in mind even now.

## 2. Community question-bank sharing (opt-in)

- A space where students can **share test banks** with each other, instead of every user
  importing in isolation.
- This is the **only** intended exception to the day-one privacy/data-isolation rule
  ([decisions.md §3.7](decisions.md)) — and it must be **explicitly opt-in**. Default stays
  private; sharing never happens silently.
- Potential upside: solves the cold-start "I have no bank yet" problem, increases stickiness,
  and (for shared/vetted banks) enables variant reuse that cuts per-user LLM cost
  (see [business.md](business.md)).
- Definitely **post-MVP** — but flagged now because the privacy architecture we build for the
  MVP should not make this impossible later (build per-user isolation in a way that can later
  grant explicit, scoped sharing — not a hard wall that has to be torn down).

## 3. Intellectual-property considerations (tied to #2)

Opening up community sharing raises IP/legal questions we must answer **before** shipping any
sharing feature:

- **Ownership of uploaded content.** Much of what users import (e.g. commercial NCLEX banks)
  may be **copyrighted by third parties** (UWorld, Archer, publishers). Letting users
  redistribute that through our platform could expose us to liability. Private personal use is
  one thing; community redistribution is a different legal posture entirely.
- **What can actually be shared?** Options to weigh later: only user-*authored* questions,
  only questions the user attests they have rights to, only ConceptLoop-*generated* variants
  (and who owns those?), or a vetted/licensed shared library.
- **Ownership of regenerated questions.** When our AI regenerates a variant from a user's
  imported question, who owns the variant — the user, us, or is it a derivative of the original
  copyrighted item? Needs a clear stance and ToS language.
- **DMCA / takedown + moderation** process for shared content.
- **Provenance/attribution** if sharing is allowed.

> Action when we revisit: get a clear legal read on third-party content redistribution before
> any sharing feature leaves the drawing board. This is a "don't build it until the IP question
> is answered" item.

---

## 4. Question bank generation as a service (B2B product surface)

A distinct product surface that reuses the same underlying generation engine: instead of
remediating *one student's misses*, generate a **fresh, varied question bank** from an
existing set + solutions — sold to instructors, content creators, or licensing bodies.

Use cases:
- Course instructors who need exam variants that can't be shared between cohorts.
- Content creators (Klimek tier) who want to expand their library without writing every question.
- Licensing bodies generating new items from a validated concept map.

This is a B2B product, separate from the student-facing loop, with potentially higher
willingness-to-pay per generation. The *creativity and quality* of the output is the product —
which creates a natural premium tier. See also the IP note in §3 (who owns generated content).

**Prerequisite:** the core remediation loop must be validated first. Don't build this until
the student-facing product has proven the generation quality is trustworthy.

---

## Parking-lot index (add as they come up)

- [ ] Cross-platform web + mobile, resume-across-devices session continuity
- [ ] Opt-in community bank sharing
- [ ] IP / copyright / ownership framework for shared & regenerated content
- [ ] Question bank generation as a B2B service (instructors, content creators, licensing bodies)
