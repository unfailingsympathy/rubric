# Long-Term Retention Design

Design spec for a verbatim scripture memorisation app. The target is not
"know it next month" — it's **retrievable in thirty years, cold, under
pressure, without a cue in the environment.** Every decision below is
subordinate to that.

---

## 0. Why verbatim text is its own problem

Most spaced-repetition design assumes semantic recall: a fact, a word
pair, a definition. Verbatim recall of fixed text is a different task
with different failure modes:

| Failure mode | What it looks like | Usual cause |
| --- | --- | --- |
| Serial-chain fragility | Fluent from word 1, helpless from the middle | Only ever practised start-to-finish |
| Reference decay | Can recite it, can't name where it is | Reference never independently tested |
| Middle sag | First and last phrases solid, middle mush | Serial position effect, untargeted |
| Fluency illusion | Feels known, isn't | Self-grading on near-misses |
| Paraphrase drift | Meaning intact, wording degraded | Scored on gist, not words |
| Context dependency | Works at your desk, fails in conversation | Single practice context |

The design exists to attack these six specifically. A generic flashcard
loop addresses none of them.

---

## 1. Scheduler: FSRS, not SM-2

**FSRS** (Free Spaced Repetition Scheduler) models memory as three
quantities per item:

- **Retrievability (R)** — probability you'd recall it right now
- **Stability (S)** — how many days until R decays to 90%
- **Difficulty (D)** — how resistant this item is to gaining stability

Reviews are scheduled when R crosses your target. This beats SM-2's
fixed ease-factor arithmetic materially — roughly 20–30% fewer reviews
for equivalent retention in Anki's own community benchmarking.

### Desired retention: set it to 0.90

This is the single most consequential number in the app, and the
instinct to crank it up is wrong.

Workload scales **steeply and non-linearly** as retention target rises:

| Target | Relative review load | Practical meaning |
| --- | --- | --- |
| 0.85 | 0.7× | Noticeably lossy; verses feel shaky |
| **0.90** | **1.0×** | **Recommended baseline** |
| 0.95 | ~2× | For a small "core" set only |
| 0.99 | ~10× | Unsustainable past ~50 verses |

Recommendation: **global 0.90, with a per-verse override to 0.95 for a
hand-picked core of 20–30 verses.** You do not need identical fidelity
across 300 verses, and pretending you do is how the collection collapses.

### Lapse handling

SM-2 resets interval to near-zero on failure. That is punitive and
wrong — some stability survives a lapse. FSRS reduces S rather than
zeroing it. **Additionally: a near-miss must not score as a failure.**
One wrong function word is not the same event as a blank stare, and
collapsing them corrupts the scheduler's model of the item.

---

## 2. Never retire a verse

The most important single finding in this literature, and the one most
apps get wrong.

Karpicke & Roediger's work on retrieval practice found that **dropping
items from practice once they've been recalled correctly devastates
long-term retention** — while continuing to *retrieve* them preserves
it. Critically, continued *re-studying* after first success does
nothing. It is retrieval, specifically, that builds durability.

**Design consequence:** there is no "learned" state. There is no
graduation. A verse at a four-year interval is still in the queue; it
just comes round rarely. The app must never offer a "mastered, remove
from rotation" action, because that action is the mechanism by which
people lose everything they memorised in their twenties.

### The permastore tier

Bahrick's long-horizon studies (Spanish vocabulary tracked over
decades) support the existence of a near-permanent retention state
reached after enough successful retrievals at long spacings. That state
is the design target, and it's reachable — but only via the long tail
of reviews, not via intensive early drilling.

Implementation: verses whose interval exceeds ~18 months move to a
**Vault** tier — still reviewed, but with a lighter verification
protocol (see §4) to keep the load honest.

---

## 3. Multiple retrieval routes

A single practice route builds a single entry point. Verbatim text
needs several, because you don't get to choose your cue in real life.

Six drill modes, each attacking a different failure mode:

1. **Reference → text.** The primary route. "Romans 8:28" → full text.
2. **Text → reference.** Attacks reference decay. Show the verse, name it.
3. **First-letter scaffold.** Every word reduced to its initial:
   `F w k t t t G a t f g t t w l H…` — the classic verbatim technique,
   and the best rung between "visible" and "blank".
4. **Progressive cloze.** Deletions weighted toward **middle chunks**,
   because primacy and recency already protect the edges. Untargeted
   random deletion wastes reps on phrases you already own.
5. **Cold random-start.** An arbitrary mid-verse phrase → continue from
   there. This is the antidote to serial-chain fragility and it is
   almost universally missing from existing apps.
6. **Theme → verse.** "Which verse addresses anxiety?" Builds the
   semantic route in, which is how you actually reach for a verse when
   you need one.

The scheduled review draws from modes 1–5 with mode weighting shifting
toward the harder modes as stability grows.

---

## 4. The drill ladder, difficulty, and success rate

Bjork's distinction between **storage strength** and **retrieval
strength** drives this: retrieval that succeeds *easily* produces
little durable gain. Retrieval that is effortful but successful
produces the most.

```
L0  Listen to it recited           (encoding — not scored, see §6)
L1  Full text, recite along        (priming)
L2  First-letter scaffold
L3  Cloze, 25% deleted, middle-weighted
L4  Cloze, 60% deleted
L5  Blank — reference only
L6  Cold random-start
```

### The tension nobody resolves cleanly

Bjork says maximise difficulty. Engelmann's Direct Instruction and
Rosenshine's synthesis of effective-teacher research both say the
opposite: keep success rates around **80–90% during instruction**, because
below that, learners stall and quit. Both bodies of evidence are strong.
They genuinely conflict, and any design that pretends otherwise is
hiding something.

The resolution I'd adopt — and it's a judgement call, not a settled
finding — splits by phase:

- **Acquisition** (first ~3 weeks of a verse): favour high success.
  Advance a rung only after two consecutive clean passes. If you fail
  twice at a rung, **drop back one.** My earlier "never drop back" rule
  was wrong; it optimises for difficulty at the expense of the success
  rate that keeps you in the chair.
- **Retention** (consolidated, interval > 1 month): favour difficulty.
  Present at the highest passed rung and stay there.

The distinction matters because desirable difficulty is a claim about
*consolidation*, not about acquisition. Making early encoding maximally
hard just makes early encoding fail.

**Vault protocol (interval > 18mo):** L5 pass required, but a
single-slip near-miss doesn't trigger a full relearn cycle — it nudges
stability down and re-queues at a few months. The point of the Vault is
to be cheap enough that you never resent it.

---

## 5. Scoring must be automatic

Self-grading is where verbatim memorisation quietly dies. People report
"close enough" on a paraphrase, the scheduler believes them, and the
wording erodes over years without ever registering as a failure.

**Word-level alignment, not string comparison.** Compute a diff between
your attempt and the text, classify each difference, then map to a grade:

| Result | Grade | Notes |
| --- | --- | --- |
| Exact, fluent, high level | Easy | |
| Exact | Good | |
| 1–2 function-word slips (*and/the/that*) | Hard | Flag the slip visually |
| Content-word error, omission, or blank | Again | |
| Correct words, wrong order | Again | Order *is* the content here |

Show the diff inline immediately. Immediate corrective feedback on a
confidently-held error is unusually effective — the hypercorrection
effect — so the moment of being wrong is a moment to exploit, not soften.

### Correct on a protocol, don't just display the diff

Direct Instruction uses a specific four-beat correction sequence, and it
outperforms "here's what you got wrong, moving on":

1. **Model** — show and voice the correct phrase in isolation
2. **Lead** — say it together, text visible
3. **Test** — they produce it alone, immediately
4. **Retest** — re-queue that verse *later in the same session*

Step 4 is the one apps skip. An error corrected and then re-tested twenty
minutes later within the same sitting is worth substantially more than
an error corrected and deferred to whenever FSRS next surfaces it. This
is a within-session mechanism sitting *underneath* the scheduler, not a
modification to it.

---

## 6. Session structure

### Listen before you produce

The two most successful verbatim-transmission systems in history — the
Quranic *hafiz* tradition and Suzuki string pedagogy — both front-load
**listening**. Suzuki's mother-tongue principle is explicit about it:
saturate the ear first, produce second. My earlier L0 was "read aloud,"
which is the wrong first move. Replace it with: hear the verse recited
correctly, several times, before your first production attempt.

This also fixes an error you otherwise bake in permanently — your first
self-generated reading establishes the phrasing you'll keep for decades,
and if it's wrong or arrhythmic, you've encoded that.

Implementation: TTS is adequate; a recording of a human reciter is better.

### Give the session a three-tier shape

The hafiz tradition splits daily work into **sabaq** (today's new
portion), **sabqi** (the last week's), and **manzil** (the whole
accumulated corpus, on a long cycle). That's a hand-rolled spaced
repetition system refined over fourteen centuries, and it maps almost
exactly onto FSRS state.

Keep FSRS doing the maths, but **use these three tiers as the interface**:

| Tier | Contents | Feel |
| --- | --- | --- |
| New | 1–2 verses | Slow, guided, careful |
| Recent | Anything under ~1 month stability | The bulk of the work |
| Vault | Everything else, long cycle | Fast, light, confidence-building |

An undifferentiated queue of 20 items is demoralising. The same 20 split
into "one new, twelve recent, seven vault" has a shape, a beginning and
an end, and a reliable easy stretch at the finish. That is a motivational
property, not a cosmetic one, and adherence is the binding constraint on
this whole project.

**Interleave within tiers, don't block.** Practising one verse ten times
feels productive and performs worse at retention than mixing verses.
Cap reps per verse per session at **2** (the §5 correction retest is the
one exception).

**Don't overlearn.** Overlearning gains decay within weeks; spacing
gains don't. Extra reps today are strictly worse than the same reps
spread over three weeks. The cap above is load-bearing.

**Say it out loud.** The production effect — vocalising rather than
reading silently — is a cheap, reliable gain. Where speech input is
available, prefer it; where not, prompt for it explicitly.

**Evening session for new material.** Sleep-dependent consolidation of
declarative material is real if modest. New verses introduced in a
last-session-before-bed slot get a free advantage.

**Vary context.** Don't build a fixed ritual. Practising in one chair
at one time of day builds context-dependent recall that fails when the
context changes. A mobile app does this naturally — don't design
against it.

---

## 7. Chunking and encoding

**Phrase units of 3–7 words, split at clause boundaries** — never at
arbitrary word counts. Grammatical structure is a scaffold; cutting
across it discards free structure.

**Bidirectional chaining.** Forward-only cumulative build (A, AB, ABC,
ABCD) over-trains the opening — which primacy already protects. Alternate
with backward chaining (D, CD, BCD, ABCD), a standard technique in music
and stage memorisation that scripture apps rarely use.

### Encode the intention, not the topic

My earlier "one-line meaning note" was too weak, and the research on how
professional actors learn lines shows why. Noice & Noice found that
actors achieve extraordinary verbatim fidelity with comparatively little
rote drilling — because they don't encode text, they encode **what the
speaker is trying to do to whom, and why**. Verbatim accuracy arrives as
a byproduct of that, because once you hold the intention, the specific
words start to feel *necessary* rather than arbitrary.

So the intake prompt is not "what is this verse about."

| Weak (topic) | Strong (intention) |
| --- | --- |
| "This verse is about grace" | "Paul is reassuring people who believe they've been abandoned" |
| "About anxiety" | "A command, given to someone visibly afraid, with a reason attached" |

Same field in the data model, much better question asked of it.

### Two independent routes, and use both

Worth noticing a real tension in the source traditions: actors get
fidelity through *meaning*, while a great many huffaz memorise the entire
Quran verbatim in a language they don't speak — fidelity through
**sound and rhythm**, with no semantic route at all.

Both plainly work. That implies two separable encoding routes —
semantic and phonological — and a design that builds both has redundancy
when one fails. Hence: intention notes (semantic) *and* consistent
recitation rhythm (phonological), not one or the other.

**Keep the prosody fixed.** Both the hafiz and yeshiva traditions use a
stable melodic or cantillation contour, and I under-rated this earlier as
"awkward to build." You don't need actual melody — you need the same
phrasing, stresses, and pauses every single time. Prosodic consistency is
nearly free to encourage and probably does most of the work that melody
does.

**Keep the visual layout fixed too.** Quintilian recommended learning
from a consistent page layout, and he was right: reflowing text destroys
a free visual anchor. Render each verse in identical fixed line breaks,
one phrase per line, every time — never wrapped to the viewport.

---

## 8. Maintenance load — the thing that actually kills projects

Reviews accumulate with collection size. Unmanaged, the daily queue
grows until it's abandoned. Three mechanisms:

1. **Daily cap** (default 20 reviews) with overflow rolled forward,
   oldest-due first.
2. **Intake throttle.** New verses are blocked while the backlog exceeds
   the cap. Enthusiasm adds verses faster than it sustains them, and the
   app should refuse rather than let you bury yourself.
3. **Absolute intake cap of 2 new verses per day**, backlog or no backlog.
   The hafiz tradition holds new intake to roughly half a page a day and
   sustains that for years; the discipline is the point. A week of
   enthusiasm at ten verses a day creates a review debt that arrives three
   weeks later and ends the project. The app should be willing to tell you
   no on a day when you feel great.
4. **Vault tier** (§4) to keep mature verses cheap.

### The steady-state arithmetic, because it matters

At 0.90 retention, a mature verse costs roughly **1–2 reviews per year**.
So:

- 100 verses → ~150 reviews/year → **under 1 per day**
- 300 verses → ~450 reviews/year → **1–2 per day**

That is the whole ongoing cost of holding 300 verses for life. Front-loaded
work is real — a new verse takes 8–12 reviews in its first year — but the
tail is nearly free. Worth knowing up front, because the intuition that
this becomes an ever-growing burden is simply false, and that false
intuition is why people don't start.

---

## 9. Evidence quality — an honest ledger

Not all of the above is equally well-supported. Building on the strong
parts and hedging the weak ones is itself part of the design.

**Strong.** Spacing effect. Testing/retrieval-practice effect. The
retrieval-vs-restudy asymmetry. FSRS outperforming SM-2 (large-scale
practical benchmarking). Automatic over self-scoring.

**Moderate.** Interleaving for verbatim text — most interleaving
research concerns category learning and motor skills, and the transfer
to fixed-text recall is plausible but not directly established.
Sleep-timing benefits. Production effect. Backward chaining (strong
practitioner track record, thin formal literature). Bloom's 2σ tutoring
figure is widely cited and widely disputed on magnitude — the direction
holds up better than the number. Intention-based encoding (§7) rests
largely on one research programme, though it fits the broader
depth-of-processing literature.

**Weak or overstated.** Memory palaces for verbatim wording — excellent
for *order* and for lists, much less so for exact phrasing; useful for
learning which references you hold, not for the words. Overlearning.
Melody as a retention aid is genuinely promising for verbatim text but
awkward to build — hence the fixed-prosody compromise in §7.

**Existence proofs rather than experiments.** The hafiz and yeshiva
traditions (§6, §12) demonstrate that these methods work at scale over
centuries, which is worth more than most lab results for a question about
decades-long retention. But they're confounded by motivation, community,
and selection, so I've borrowed their *structures* while grounding the
mechanisms elsewhere.

**Discard.** Anything learning-styles-shaped. Passive re-reading.
Highlighting.

> **On citations:** I've named findings and researchers from memory and
> I have no search access here, so treat specific attributions as
> pointers to look up rather than verified references — I can get names,
> dates, and effect sizes wrong. The design decisions stand on their own
> merits; the labels are for your own further reading.

---

## 10. Data model implication

Per verse, persist: reference, text, phrase chunks, meaning note,
FSRS state (S, D, last review, due date), drill level, per-mode pass
counts, lapse history, tier (Active / Vault), retention override.

Per-mode pass counts matter — a verse can be solid on reference→text
and hollow on cold-start, and a single aggregate score hides exactly
the weakness you most need to see.

---

## 11. Teaching the method, not just running it

Everything above describes an app that *schedules* you. It leaves you
dependent: the method lives in the software, and if the phone dies or
the repo goes stale or you lose interest for a year, nothing transfers.
The stronger goal is that after a hundred verses **you can memorise
without the app** — and could teach someone else.

That is an instructional-design problem, and it has a known shape.

### The frame: cognitive apprenticeship

Collins, Brown & Newman's sequence is the right skeleton:
**model → coach → scaffold → fade → articulate → reflect.** Most
learning apps do the first three and stop, which produces users who are
good at the app.

The fading is not politeness. The **expertise reversal effect** says
support that helps a novice actively *degrades* an expert's
performance — so scaffolding you never remove ends up making you worse
than no scaffolding at all. Fading is a functional requirement.

### Six features that do the teaching

**1. A worked example as your first verse.**
Not a tutorial screen — an actual verse you keep. Fully narrated: here
is the clause boundary I'm splitting on and why, here is the meaning
note, here is why we're at first-letters and not blank yet. The
worked-example effect is one of the better-established findings in
instructional design, and five minutes here beats any amount of
onboarding copy.

**2. An intake coach that fades on a schedule.**

| Verses | Chunking behaviour |
| --- | --- |
| 1–5 | App splits, and explains each decision |
| 6–15 | App proposes, you edit, app comments on your edit |
| 16–30 | You split, app silent unless something's off (chunk >9 words, split mid-clause) |
| 31+ | Fully yours; app never volunteers |

By verse thirty, chunking is a skill you have rather than a service you
receive.

**3. Just-in-time micro-explanations, shown exactly once.**
When cold random-start first appears, one screen: what this mode is
for, which failure mode it attacks. Then never again — available behind
a "why this?" affordance, never re-pushed. Nobody reads onboarding;
everybody reads an explanation at the moment they're confused.

**4. The calibration loop — the single most valuable teaching device here.**
Before a long-interval review, ask: *"think you'll get this?"* Yes/no.
Record prediction against outcome. Then surface it:

> You predicted you'd recall 94% of these. You recalled 71%.
> Your confidence runs about 20 points ahead of your memory.

Judgments of learning are systematically miscalibrated, and *being told
that* changes nothing. Being shown it, about your own memory, with your
own numbers, changes a great deal. It also retroactively justifies every
other design decision in this document — the automatic scoring, the
refusal to retire verses, the 0.90 target — because you can now see why
trusting your sense of "I know this" would have failed you.

**5. Diagnosis instead of scoring.**
Don't just mark a miss. Name the pattern:

> Third miss in the middle chunk. That's the serial position effect —
> your edges are protected, the middle isn't. Drill it isolated.

Do that fifty times and the user starts diagnosing their own failures,
which is most of what expertise in this domain actually is.

**6. Graduation drills.**
Periodically, the app gives a reference and *withholds all support*:
memorise it offline, using the method, then come back and type it cold.
Transfer gets tested rather than assumed. This is the only feature that
directly measures whether the teaching worked.

### The constraint that governs all six

**Teaching must never sit between the user and their daily reviews.**
Every explanation screen is an opportunity to quit, and adherence is
worth more than pedagogy — an app you use daily and understand
shallowly beats an app you understand deeply and abandoned in week
three. So: explanations happen at intake or *after* a session, never as
a gate before drilling. And the whole teaching layer needs an off
switch, both for expertise reversal and because some users just want
the drill.

### One page that outlives the app

Finally: a printable one-page method card — chunking rules, the drill
ladder, the interval schedule, the failure modes and their fixes. The
app is a scaffold for the method, and scaffolds come down. Assume this
software will eventually stop working and make sure that doesn't cost
you the skill.

---

## 12. The missing layer: reciting to a person

This is the largest gap in everything above, and it took auditing the
traditions rather than the lab literature to see it.

Look at what the genuinely successful verbatim systems have in common:

| Tradition | Core mechanism |
| --- | --- |
| Quranic *hafiz* | *Tasmi'* — you recite daily **to a teacher** who corrects on the spot |
| Yeshiva study | *Chavruta* — paired study, partner catches your errors aloud |
| Suzuki | Weekly lesson plus a parent present at home practice |
| Bloom's tutoring result | One-to-one instruction, ~2σ over conventional teaching |
| Ericsson's deliberate practice | A coach supplying immediate external feedback |

Every one of them is **social and oral**. The design above is solitary
and largely silent. That is not a small omission — it may be the single
biggest predictor of whether someone still holds their verses in ten
years, and no scheduling algorithm compensates for it.

An app cannot be a teacher. It can, however, refuse to pretend the
teacher is unnecessary, and it can lower the friction of finding one.

**What to build:**

1. **Listener mode.** Hand your phone to someone. The verse text shows on
   their screen with tappable words; you recite aloud; they tap anything
   you miss. Thirty seconds of setup, and it converts any willing person
   in the room into a *tasmi'* partner with no training required.
2. **Record and self-listen.** When no one's available, record the
   recitation and play it back against the text. Weaker than a human, far
   better than silent typing — and hearing your own error is closer to
   being corrected than reading a diff is.
3. **A weekly oral milestone.** One verse a week recited to an actual
   person, tracked as a streak of its own. Deliberately not substitutable
   by app-only work.
4. **Partner sync (optional).** Two people on the same verse list, each
   able to see the other's due queue. This is the *chavruta* pattern and
   it is the cheapest adherence mechanism in existence — the thin version
   is one shared JSON file in a Gist.

**Honest framing for the UI:** the app should say plainly, once, that
solo use is the weakest version of the method. Not as a nag — as
information the user deserves before they invest three years in it.

### Note on evidence quality here

The traditions in that table are strong existence proofs — the hafiz
system in particular has produced verbatim retention at scale for
fourteen centuries, which no laboratory paradigm can match for duration.
But they are not controlled comparisons. The social element is entangled
with selection effects, religious motivation, community structure, and
full-time schedules. Bloom's tutoring result and the deliberate-practice
literature are better-controlled and point the same way, which is why I'd
act on it — but "reciting to a person is worth more than any feature in
§1–§11" is my judgement from converging weak evidence, not an established
effect size.
