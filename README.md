# The Unofficial Guide

Guanwen Wang, city_guides

> **This file is your submission.** Fill it in as you go — most sections get
> written during the milestone that produces them, not at the end.
>
> How the starter works, and every command you'll need, is in `RUNNING.md`.
> Leave that file alone.
>
> **Paste everything as text.** No screenshots, no video. A typed table gets
> full credit; a picture of the same table gets none.
>
> Delete these instruction blocks as you replace them. The `<!-- -->` comments
> are notes to you and don't show up when the page renders — you can leave them
> or remove them.

---

# Unit 1

## What This Does

This is a retrieval-augmented question answering system over a corpus of travel
guides (`city_guides`), covering 14 towns. A user asks a question
— for example, "How often do trams run in Marchwood on weekdays?" — and the
system retrieves the most relevant chunks from the guides, checks that they
are close enough to be useful, and generates an answer that cites the specific
file and section it came from. Questions the corpus does not cover are refused
with "I don't have enough information about that" rather than answered from the model's
training data.

## Chunking Strategy

**Chunk size:** 800 characters
**Overlap:** 100 characters, only when a section exceeds 800 characters.

I read three `city_guides` documents (`guide_seasons.md`,
`guide_accessibility.md`, `guide_elder_ness.md`) and found they all share
the same structure: a `#` title followed by multiple `##` sections, each
covering one self-contained topic (one season, one accessibility category,
one aspect of a town).

The starter chunker cut these on a fixed 800-character window and produced:
- A chunk ending mid-sentence: "...The station is a 15-"
- A 24-character chunk (the tail of a document that didn't divide evenly)
- Chunks that mixed two `##` sections, so a question about one topic
  matched a chunk mostly about another

My splitter uses `##` headings as the primary boundary. Each section
becomes one chunk. If a section exceeds 800 characters, it is split
further on paragraph breaks with 100 characters of overlap so no sentence
is cut in half. Each chunk carries a `[file > section]` header so the
source line can cite the specific section, not just the file.

**Result after re-indexing:** 98 chunks (was 51, +92%), average 331 characters
(was 650, -49%), shortest 50 (was 24, +108%), longest 754 (was 800, -6%). The shortest chunk
is the intro to `guide_accessibility.md` — a short but self-contained
opening paragraph, not a fragment.

## Sample Chunks

**Chunk 1** — source: `guide_accessibility.md#0` — produced by: `chunker.py::split_documents`

```
[guide_accessibility.md > intro]
# Getting around the region with limited mobility

An honest assessment rather than a promotional one. Some of these places are
difficult and it is better to know in advance.
```

**Chunk 2** — source: `guide_corry_vale.md#6` — produced by: `chunker.py::split_documents`

```
[guide_corry_vale.md > When to go]
## When to go

May to September. Outside those months the pub in the third village closes, the farm shop reduces its hours, and several footpaths become genuinely boggy rather than merely wet. The road is not gritted above the second village and is impassable in snow.
```

**Chunk 3** — source: `guide_givens_mill.md#3` — produced by: `chunker.py::split_documents`

```
[guide_givens_mill.md > Eat and drink]
## Eat and drink

A tearoom attached to the mill, open 10 to 4 daily except Tuesdays, which sells bread made from the flour ground twenty metres away and is the reason most people come. One pub, food served lunchtimes and Thursday to Saturday evenings.
```

**Chunk 4** — source: `guide_kestrelford.md#6` — produced by: `chunker.py::split_documents`

```
[guide_kestrelford.md > When to go]
## When to go

Late spring and early autumn. The Saturday market runs year-round but is much reduced from November to February. August is busy with walkers. The single-track approach road is genuinely difficult in snow and the town can be cut off for a day or two most winters.
```

**Chunk 5** — source: `guide_regional_transport.md#1` — produced by: `chunker.py::split_documents`

```
[guide_regional_transport.md > The railway]
## The railway

The line runs along the river valley, connecting Brightwater to the regional
hub in 50 minutes. Eleven services a day on weekdays, six on Sundays. The line
north of Brightwater closed in 1963 and everything beyond it is bus or car.

Tickets are cheaper booked the day before than on the day, and considerably
cheaper than that booked a week ahead. There is no ticket office at
Brightwater station outside weekday mornings; the machine on the platform takes
cards only.
```

## Sample Answer

**Question:** What is the population of Elder Ness?

**Answer:**

```
The population of Elder Ness is 300, according to guide_elder_ness.md.

Sources retrieved: guide_eating.md, guide_elder_ness.md
```

**My relevance cutoff:** 0.60

I ran five questions my corpus covers and the five in OUT_OF_SCOPE that it 
clearly doesn't, and wrote down the best distance for each. The two groups
separated clearly with no overlap, the gap is between 0.458 - 0.758

| Question | In corpus? | Best distance |
|---|---|---|
| How often do trams run in Marchwood on weekdays? | yes | 0.288 |
| Which town is easiest with limited mobility? | yes | 0.458 |
| How long is the walk to the Elder Ness lighthouse? | yes | 0.255 |
| What is the population of Elder Ness? | yes | 0.327 |
| What time should you arrive at Halden Bay in August? | yes | 0.265 |
| What is the capital of Mongolia? | no | 0.758 |
| How do I change the oil in a diesel engine? | no | 0.923 |
| Who won the 1994 World Cup? | no | 0.998 |
| What is the recommended dosage of ibuprofen? | no | 0.792 |
| How do I write a for loop in Rust? | no | 0.854 |

The in-scope group ranged 0.255–0.458; the out-of-scope group ranged
0.758–0.998. The gap is 0.300 wide. I kept the starter's cutoff of 0.60,
which sits near the middle of the gap: 0.14 above the highest in-scope
distance and 0.16 below the lowest out-of-scope distance. Every in-scope
question passes; every out-of-scope question is refused.

## How I Used AI

<!-- Two specific moments. For each: what you asked for, what came back, and
     what you changed about it.

     "I asked Claude to write the chunking function from my notes. It ignored
     the overlap, so I added that myself" is the level of detail we're after.
     "I used AI to help me code" is not.

     Milestone 5. -->
**1. Chunking strategy — from a generic suggestion to a corpus-specific one.**

I asked Claude to suggest a chunking strategy. The first suggestion it gave me was generic: split
on paragraph breaks. I read three of the actual documents and saw that every one
of them is structured as a `#` title followed by multiple `##` sections,
and that each `##` section is already a self-contained topic. So I
changed the strategy with what I found. The result was
98 chunks (was 51 under the starter) with an average length of 331
characters, and the two previously broken chunks ("The station is a 15-"
and a 24-character tail) both disappeared.

**2. Relevance cutoff — using AI to analyse the gap, not to pick the number.**

I ran all 5 in-scope and out-of-scope questions and recorded the best distances,
then asked where the gap was and how to decide the cut-off.
It showed me the in-scope group ranged 0.255–0.458 and the out-of-scope group
0.758–0.998, with a 0.300-wide gap and no overlap. It suggested placing the cutoff near the midpoint
(~0.608). I kept the starter's 0.60 instead, because it already sits in
the middle of the gap, and keeping the starter value is itself
evidence that my chunking strategy separated the two groups cleanly, and
because a round number is easier to explain and reproduce than a fitted
one. All five in-scope questions pass and all five out-of-scope questions
are refused.

<!-- ── Stretch features ─────────────────────────────────────────────────────
     Doing one? Say so here BEFORE you start. A feature this README never
     claims earns nothing.
     ───────────────────────────────────────────────────────────────────────── -->

---

# Unit 2

<!-- These sections get ADDED to what's already above. Don't delete or rewrite
     unit 1 — the point is that someone can see what you said before you knew
     how it went. -->

## Run Log — Before

Three runs per criterion, caching off. Produced by `run_eval.py::main`,
stored in `results/run_2026-09-26_1808_before.md`.

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 2. Every answer names a source | 5 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 3. Gate stops out-of-corpus questions | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 4. No chunk ends mid-sentence | 5 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 5. Source names a specific file | 3 of 3 | 5/5 | 5/5 | 5/5 | MET |

Criteria 3 and 4 are deterministic — retrieval is a comparison against a
fixed number and chunk boundaries do not change between runs — so the
same number goes in all three columns. Criteria 1, 2 and 5 depend on the
generated answer and were measured three times.

### Real output

Produced by `run_eval.py::main`, stored in
`results/run_2026-09-26_1808_before.md`.

**Criterion 1 — run 1:**

> Question: How often do trams run in Marchwood on weekdays?
> Best distance: 0.3251 (passed the gate)
>
> ```
> In Marchwood, trams run every 8 minutes on weekdays.
> Source: `guide_marchwood.md`
> ```

**Criterion 1 — run 1 (Elder Ness population):**

> Question: What is the population of Elder Ness?
> Best distance: 0.3717 (passed the gate)
>
> ```
> The population of Elder Ness is 300 (guide_elder_ness.md).
> ```

**Criterion 2 — run 1:**

> Question: What time should you arrive at Halden Bay in August?
> Best distance: 0.3505 (passed the gate)
>
> ```
> If you are going to Halden Bay in August, you should arrive before 10am.
> Source: `guide_seasons.md` (also mentioned in `guide_regional_transport.md`
> and `guide_halden_bay.md`).
> ```

**Criterion 3 — deterministic, one pass:**

| Out-of-scope question | Best distance | Gate |
|---|---|---|
| What is the capital of Mongolia? | 0.887 | refused |
| How do I change the oil in a diesel engine? | 0.897 | refused |
| Who won the 1994 World Cup? | 0.903 | refused |
| What is the recommended dosage of ibuprofen for a headache? | 0.829 | refused |
| How do I write a for loop in Rust? | 0.853 | refused |

Gate refused 5 of 5.

**Criterion 4 — sample of 5 chunks, produced by `chunker.py::split_documents`:**

```
guide_halden_bay.md#4: ends with '...covers the fishing industry and takes 40 minutes.'
guide_givens_mill.md#3: ends with '...food served lunchtimes and Thursday to Saturday evenings.'
guide_marchwood.md#1: ends with '...costs more than the equivalent taxi shared between three people.'
guide_brightwater.md#3: ends with '...earlier than visitors expect, and on Sundays many close entirely.'
guide_eating.md#2: ends with '...the hardest meal to find anywhere except Marchwood and Thornby Wells.'
```

All 5 end with a complete sentence.

**Criterion 5 — run 1:**

> Question: Which town in the region is the easiest to get around with limited mobility?
> Answer:
>
> ```
> **Thornby Wells** is the easiest town in the region to get around with
> limited mobility because it is flat, compact, and everything is within
> three minutes of everything else.
> Source: `guide_accessibility.md`
> ```

## Verdicts

<!-- MET or MISSED for each of the five, against the target you wrote last
     unit — not a new one. Plus a sentence on how you decided. That sentence
     matters most where it was close.

     If your target said 4 of 5 and your runs came out 4, 3, 4, that's a MISS.
     The target has to hold, not show up occasionally.

     Milestone 2. -->

| # | Criterion | Verdict | How I decided |
|---|---|---|---|
| 1 |  |  |  |
| 2 |  |  |  |
| 3 |  |  |  |
| 4 |  |  |  |
| 5 |  |  |  |

## Diagnoses

<!-- For each miss: which stage caused it, and how. The stage alone isn't
     enough — you need the mechanism.

     Not a diagnosis: "Question 3 didn't work."
     A diagnosis:     "Question 3 asks about laundry costs. The answer is in
                       one sentence that got split across two chunks, so
                       neither chunk on its own contains it."

     The five stages: loading → chunking → embedding → retrieval → generation.

     Look for a pattern. If three misses all ask about numbers, that's one
     problem, not three.

     Missed nothing? Say so, then say honestly whether your targets were set
     low, and which one you'd tighten and to what.

     Milestone 3. -->

## The Improvement

**What I changed:**

**Why I picked it:**

<!-- Connect it to a specific diagnosis above in one sentence. If you can't,
     you picked a fix because it sounded impressive. -->

### Run Log — After

<!-- Same format, same five criteria, three runs each.
     `python run_eval.py --label after` -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 |  |  |  |  |
| 2. Every answer names a source | 5 of 5 |  |  |  |  |
| 3. Gate stops out-of-corpus questions | 4 of 5 |  |  |  |  |
| 4. | | | | | |
| 5. | | | | | |

**Did it help?**

<!-- Say plainly whether it did, and how you know. If it made things worse,
     say that — a change that backfired, honestly reported, earns full credit
     and is more interesting than one that worked. What matters is that you can
     tell.

     Milestone 4. -->

## What's Still Broken

<!-- For each criterion still missed after your fix: what you'd do about it,
     and why you stopped where you did.

     "I ran out of time" is fine if it's true. Pretending nothing is left is
     not.

     Milestone 5. -->

## What I'd Do Differently

<!-- Knowing what you know now — which of your five criteria would you write
     differently, and why?

     Milestone 5. -->
