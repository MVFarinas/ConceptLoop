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

## Secondary / expansion markets (later, same playbook)

Once the loop is proven on NCLEX, the *exact same mechanic* transfers to any high-stakes,
MCQ-heavy, motivated-learner exam:

1. **USMLE / medical licensing** — adjacent, same format, even higher stakes.
2. **Bar exam / law (MBE)** — MCQ-heavy, high willingness-to-pay, nameable doctrinal confusions.
3. **Professional certifications** — CPA, actuarial, engineering (FE/PE), PMP, cybersecurity, etc.
4. **Standardized admissions tests** — MCAT, LSAT logical reasoning (with care).

We expand by *adding domains*, not by changing the product — each new vertical brings its own
user-supplied banks and answer keys.

## Who we explicitly do NOT target first

- K–12 / casual learners — low willingness-to-pay, low-stakes, weak motivation to do cross-off.
- "Generate me a quiz from my notes" users — out of scope (see [decisions.md](decisions.md)).
- Open-ended / essay / non-MCQ formats — the cross-off mechanic doesn't apply.

## Pilot testing candidates

**Strongest near-term option: University of Alberta nursing students.**
The project owner is at UAlberta (`@ualberta.ca`), which gives direct, low-cost access to a
real cohort of the exact primary demographic.

Concrete pilot paths to pursue:
- **UAlberta Faculty of Nursing** — recruit a small group of students currently in NCLEX/exam
  prep; have them run their existing practice banks through ConceptLoop.
- **Nursing student associations / study groups** — informal pilots, fast feedback loops.
- **A single instructor or course** willing to let students try it for one unit's worth of
  practice questions (mini B2B2C dry run, and a source of a vetted bank).
- **Personal network** — any nursing students/recent grads for 1:1 think-aloud sessions while
  they hit the 50/50 moment (qualitative gold for tuning misconception inference).

**What we want to learn from the pilot:**
1. Do students actually use the cross-off interaction, or skip it? (Validates the keystone signal.)
2. When the concept comes back as a *regenerated* question, do they feel it tested
   understanding better than a re-shown original? (Validates the core hypothesis.)
3. How accurate/useful is the inferred misconception, judged by the student? (Validates the core IP.)
4. What do confident students who skip cross-off and still miss want to happen? (Resolves Open Question 4.1.)
