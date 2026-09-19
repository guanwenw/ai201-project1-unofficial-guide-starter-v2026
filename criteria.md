# Acceptance criteria — The Unofficial Guide

Five criteria that say what "working" means for this system, written in unit 1
**before** any results existed.

An acceptance criterion names a target: a number, a count, a rate, or something
a person could plainly observe. *"Retrieval works"* is an opinion. *"For at
least 4 of my 5 test questions, the top results include a chunk containing the
answer"* is a criterion.

Under each one, write a sentence or two on **why that target** and not a
stricter or looser one. A reason that says something about your corpus or your
pipeline earns credit; *"80% seemed reasonable"* does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

---

## 1. Retrieved chunks contain the answer

For at least 4 of my 5 test questions, the retrieved chunks include one that
contains the answer.

**Why this target:**
One of my five questions — "What is the population of
Elder Ness?" — depends on a single number that appears once in the whole
corpus. If the chunk boundary falls badly, that number could end up in a
chunk that retrieval doesn't return, even though the answer exists. The
other four questions are anchored in short, self-contained sections, so
4 of 5 is the honest target. 5 of 5 would be a claim I can't defend before
seeing results.

---

## 2. Every answer names a source

Every answer the system produces names at least one source document.

**Why this target:**
Grounding is the whole point of the project — an answer
without a source is just a model's guess. My pipeline always retrieves at
least one chunk before the model runs, and the prompt requires the model to
cite what it used. The only way this fails is if the model ignores the
prompt, which is exactly what I want to catch. A target of 4 of 5 would let
one uncited answer through, and one is too many for a system whose value is
verifiability.

---

## 3. The relevance gate stops out-of-corpus questions

When I ask a question my documents clearly don't cover, the relevance gate
stops it and the system returns "I don't have enough information about that" —
in at least 4 of 5 tries.

**Why this target:**
When I ran the starter on city_guides, "what is the name of the city?" returned
best distance 0.688, above the 0.6 cutoff, and the gate correctly refused. The
five OUT_OF_SCOPE questions are all from entirely different domains, so their
distances should be well above any in-corpus question. I pick 4 of 5 rather
than 5 of 5 because one question could land just under the cutoff by
coincidence — distance is a heuristic, not a proof — and I'd rather report an
honest miss than pretend the gate is perfect.

---

## 4. No chunk ends mid-sentence

No chunk in my index ends mid-sentence. When I sample 5 chunks at random,
all 5 end with a complete sentence or a section boundary.

**Why this target:**
In Milestone 1 I saw the starter chunker produce a chunk ending in "The
station is a 15-" — a clear mid-sentence cut — and a 24-character chunk
that was just a document tail. My chunking strategy splits on Markdown
headings, so every chunk should end at a `##` boundary or a paragraph end.
Any mid-sentence ending would mean my strategy failed. I pick 5 of 5 rather
than 4 of 5 because this is a mechanical property of the splitter, not a
quality judgment — there is no reason to allow even one failure.

---

## 5. Sources name a specific file

Every answer's source line names a specific file (e.g. guide_seasons.md),
not a generic reference like "the documents". When I check 3 answers,
all 3 name a file I can open.

**Why this target:**
I care about this because grounding is only useful if a
reader can verify it. A source line saying "the documents" is unverifiable —
the reader can't tell which file actually contained the answer. My chunks
carry a header prefix with the file and section name, so the model has the
information it needs to cite precisely. I pick 3 of 3 because this is a
formatting requirement, not a retrieval quality question — there is no
reason the system should ever fail to name a file it retrieved from.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 2 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath it, like this:

         ## 1. Retrieved chunks contain the answer

         For at least 4 of my 5 test questions, the retrieved chunks include
         one that contains the answer.

         **Why this target:** ...

         > **Revised in unit 2:** For at least 4 of 5 questions, the top three
         > results contain the answer.
         >
         > **Why revised:** I couldn't judge "the chunks include one that
         > contains the answer" the same way twice — I scored two questions
         > differently on Monday than on Wednesday. The new version is
         > something I can actually check.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "I said 4 of 5 but got 2 of 5, so 2 of 5 is more realistic."

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.

     The whole reason the originals stay visible is so someone can see what you
     said before you knew the answer.
     ───────────────────────────────────────────────────────────────────────── -->
