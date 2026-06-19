# Target Demographic

## Primary wedge: nursing students prepping for the NCLEX

This is the beachhead, and it's deliberate — not just "a big market."

**Why NCLEX nursing students specifically:**

- **High-stakes + high-anxiety.** The NCLEX gatekeeps a career. Students are highly motivated
  and already pay for prep (UWorld, Archer) — willingness-to-pay is proven.
- **The "remediation" vocabulary already exists.** Students and platforms already use the word;
  we don't have to teach the category, just deliver the substance behind it.
- **Question format is uniform.** Single-best-answer MCQs dominate — perfect for the narrow
  MVP (one domain, one format) and for the cross-off interaction.
- **Misconceptions are nameable and bounded.** "Stuck between potassium and calcium" is a
  concrete, recurring, well-documented confusion — ideal for misconception inference.
- **Vetted banks exist.** In a B2B2C motion, nursing programs can supply pre-validated banks
  and answer keys, which directly satisfies our "users provide ground truth" safety requirement.
- **The 50/50 pain is acute here.** Tricky wording and close distractors are a hallmark of
  NCLEX-style items — exactly the scenario in [scenario.md](scenario.md).

## Secondary / expansion markets (same playbook, different domains)

The product is domain-agnostic. Once the loop is proven on NCLEX, the same mechanic
transfers — each vertical brings its own user-supplied banks and answer keys:

| Segment | Why it fits | Notes |
|---|---|---|
| **USMLE / medical licensing** | Adjacent to NCLEX; same format, even higher stakes. | Natural second domain. |
| **Bar exam / MBE** | MCQ-heavy; high WTP; doctrinal confusions are concrete and nameable. | |
| **University students (any subject)** | Bio, chemistry, law, arts, history — any MCQ-heavy course. | Lower stakes per exam but enormous volume; strong general B2C play. |
| **DSA / coding interview prep** | CS grads and new grads reviewing data structures & algorithms for technical interviews. MCQ-style concept quizzes fit. | High-intent, tech-savvy early adopters; comfortable with new tools. |
| **Industry re-skilling** | Experienced professionals who've been away from fundamentals — a developer who hasn't touched algorithms in years, someone re-entering a field after a break. | Strong use case for the temporal decay model (§6 of [inference.md](inference.md)): resolving misconceptions that have *re-emerged* after a gap. |
| **Professional certifications** | CPA, actuarial, FE/PE, PMP, cybersecurity (CompTIA, CISSP), etc. | |
| **MCAT / LSAT** | Admissions tests with MCQ-heavy sections. | With care — higher regulatory sensitivity. |

We expand by *adding domains*, not by changing the product.

## Who we explicitly do NOT target first

- K–12 / casual learners — low willingness-to-pay, low-stakes, weak motivation to do cross-off.
- "Generate me a quiz from my notes" users — out of scope (see [decisions.md](decisions.md)).
- Open-ended / essay / non-MCQ formats — the cross-off mechanic doesn't apply.

## Pilot testing candidates

**First user: project owner's sister**, currently studying for the NCLEX. Zero recruitment
friction, high motivation, direct feedback access. Ideal for the very first qualitative pass
before any broader pilot.

**Second wave: UAlberta nursing students** — direct institutional access via the project
owner's UAlberta affiliation (`@ualberta.ca`).

Concrete pilot paths, in order:
1. **Sister (NCLEX prep)** — first live user; qualitative think-aloud on the 50/50 moment.
2. **UAlberta Faculty of Nursing** — small cohort in active NCLEX prep; run their existing banks through ConceptLoop.
3. **Nursing student associations / study groups** — informal pilots, fast feedback loops.
4. **Friends / network in tech** — CS grads, industry professionals brushing up on DSA — validates the "re-skilling" use case and the temporal decay model.
5. **A single instructor or course** — willing to let students try it for one unit (mini B2B2C dry run and a source of a vetted bank).

**What we want to learn from the pilot:**
1. Do students actually use the cross-off interaction, or skip it? (Validates the keystone signal.)
2. When the concept comes back as a *regenerated* question, do they feel it tested understanding better than a re-shown original? (Validates the core hypothesis.)
3. How accurate/useful is the inferred misconception, judged by the student? (Validates the core IP.)
4. What do confident students who skip cross-off and still miss want to happen? (Resolves Open Question 4.1.)
5. Do returning users (gap of weeks/months) notice that the system caught a re-emerging gap? (Validates temporal decay.)
