# The Unofficial Guide

**Author:** Matthew Chao  
**Corpus:** `advice_threads`

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

I am using the advice_threads corpus.
This corpus is good for questions from students who want to prepare for whatever comes ahead - whether big decisions about majors or when to do their laundry.
The corpus is Reddit-like and has answers with their vote counts, so that you can get a sense of whether people generally agree with certain answers or not.

## Chunking Strategy

**Chunk size:** Dynamic / reply-level (one thread question + one complete response).
**Overlap:** Structural context overlap (the thread question is prepended to every reply; 0 sliding-window character overlap).

### Rationale
The `advice_threads` corpus consists of forum-style discussions where each document contains a main question (`THREAD: ...`) followed by multiple student responses with vote counts. 
Fixed-character windowing fails here: it cuts across sentences mid-word, produces degenerate tail fragments when documents don't divide evenly, and blends multiple unrelated responses into one chunk.
### Strategy Rules
1. **1:1 Reply-to-Chunk Mapping:** For each document with $N$ replies, exactly $N$ chunks are produced.
2. **Context-Prefixed Content:** Every chunk combines the thread title/question with exactly one complete reply, including its vote count header.
3. **No Fragmentation:** Chunk boundaries align with natural document sections, ensuring every chunk reads as a complete, standalone thought (satisfying Criterion 4).

## Sample Chunks

======================================================================
Chunk 1  |  source: thread_bike_commute.txt#0  |  produced by: chunker.py::split_documents
======================================================================
THREAD: Is a bike worth it for a 20 minute walk commute?

--- reply 1 (14 votes) ---
Yeah. Cuts an 18 minute walk to about 6. The thing nobody mentions is storage — covered bike parking exists at three buildings and is full by 9am at all three.

======================================================================
Chunk 2  |  source: thread_first_gen.txt#1  |  produced by: chunker.py::split_documents
======================================================================
THREAD: Anything specific for first-generation students?

--- reply 2 (41 votes) ---
The thing I'd say: the unwritten rules are the hard part, not the coursework. Ask about the unwritten rules explicitly. People are happy to explain them and nobody volunteers them.

======================================================================
Chunk 3  |  source: thread_laptop_specs.txt#2  |  produced by: chunker.py::split_documents
======================================================================
THREAD: How much laptop do I actually need for CS courses?

--- reply 3 (12 votes) ---
I did two years on an 8GB machine and it was fine until the last project, at which point it very much wasn't. 16 is the answer.

======================================================================
Chunk 4  |  source: thread_parking.txt#1  |  produced by: chunker.py::split_documents
======================================================================
THREAD: Worth getting a parking permit?

--- reply 2 (21 votes) ---
Street parking on Verrill is legal and free and unmarked, which is why half the upper years do it.

======================================================================
Chunk 5  |  source: thread_sleep_schedule.txt#1  |  produced by: chunker.py::split_documents
======================================================================
THREAD: Everyone says fix your sleep. Does it actually matter?

--- reply 2 (37 votes) ---
The library being open until 2am is a trap. It's a resource, not a schedule.

## Sample Answer

<!-- One complete question and answer, pasted as text, with the source line
     visible. Milestone 4. -->

**Question:**
What's the last date to declare a class pass/fail

**Answer:**
The last date to declare a class pass/fail is week eight (*thread_pass_fail.txt* and *thread_first_year_regret.txt*).

Sources retrieved: thread_first_year_regret.txt, thread_pass_fail.txt


**My relevance cutoff:** 0.65
The in-corpus questions had best distances ranging between 0.3285 and 0.6063. The out-of-scope questions had best distances ranging between 0.8075 and 0.8964. 
There is a clear gap between 0.61 and 0.80. The starter's default threshold of 0.60 was slightly too strict because it barely filtered out the sick-day exam question (0.6063). Placing the cutoff at 0.65 cleanly sits inside the gap: all 5 in-corpus questions are accepted, while all 5 out-of-scope questions are refused.

| Question | In corpus? | Best distance |
|---|---|---|
| Any strategies to save money on textbooks | Yes | 0.5015 |
| The best secret study spot? | Yes | 0.4341 |
| What's the last date to declare a class pass/fail | Yes | 0.3285 |
| What do I do if I get sick the day of an exam? | Yes | 0.6063 |
| What are office hours usually like | Yes | 0.4671 |
| What is the capital of Mongolia? | No | 0.8935 |
| How do I change the oil in a diesel engine? | No | 0.8964 |
| Who won the 1994 World Cup? | No | 0.8934 |
| What is the recommended dosage of ibuprofen for a headache? | No | 0.8075 |
| How do I write a for loop in Rust? | No | 0.8348 |

## How I Used AI

**1. Pressure-testing and refining acceptance criteria (Milestone 2)**
- **What I asked for:** I asked the AI to review my justifications for my five acceptance criteria in `criteria.md` against the assignment rubric.
- **What came back:** The AI pointed out that my original justifications explained why the features themselves were desirable in general, rather than justifying the specific numerical targets (e.g. why 4 of 5 instead of 5 of 5, or why at least 1 of 5).
- **What I changed:** I rewrote the justifications across `criteria.md` to ground them in my actual test questions and corpus structure — specifically noting that some test questions use phrasing that doesn't appear directly in the threads (like "save money" or "secret" study spot) which could cause semantic misses, and explaining why opinion-based threads warrant targeting at least one alternative answer.

**2. Designing and implementing the custom chunking strategy (Milestone 3)**
- **What I asked for:** I wanted each chunk to preserve the thread's original question so answers wouldn't lose their meaning (Criterion 4), and asked if I was limited to fixed window/overlap sizes or could pair questions with specific responses.
- **What came back:** The AI confirmed I could write custom Python logic to pair the thread title with individual replies, but initially proposed a strategy description for the README that hardcoded specific character counts (132 min, 281 max) from the current sample.
- **What I changed:** I rejected hardcoding those sample-specific counts into the README, insisting that the strategy description remain general and algorithmic (defining a 1:1 reply-to-chunk mapping with structural title overlap). I then had the AI implement this, eliminating naive window slicing and degenerate tail fragments.

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

<!-- Your five criteria, three runs each. `python run_eval.py --label before`
     runs the questions, puts the OUT_OF_SCOPE ones through the gate, and
     writes it all into results/ for you. Targets come from criteria.md; the
     verdict column is your call.

     Criterion 3 is measured in one deterministic pass rather than three, so
     the same number goes in all three run columns. That's correct, not lazy.

     Milestone 1. -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 |  |  |  |  |
| 2. Every answer names a source | 5 of 5 |  |  |  |  |
| 3. Gate stops out-of-corpus questions | 4 of 5 |  |  |  |  |
| 4. | | | | | |
| 5. | | | | | |

<!-- Underneath, paste the REAL output for each criterion from one of your
     runs — the actual text your system produced, not a description of it.
     Name the file and function that produced it. -->

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
