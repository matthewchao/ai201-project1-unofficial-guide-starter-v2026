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

**3. Diagnosing pipeline failure patterns and testing fixes (Unit 2)**
- **What I asked for:** I asked the AI to analyze why Question 4 dropped its source citation on Run 2, and what commands would trace the question through each pipeline stage.
- **What came back:** The AI helped trace Question 4 across the pipeline, showing that retrieval was deterministic while generation was stochastic across runs. It pointed out the conflict in the prompt between refusing when information is missing and citing a source document.
- **What I changed:** Instead of changing the question or the recommended action of tweaking the gate cutoff (which would have still returned a canned refusal without a source citation), I updated the grounding prompt in `generate.py` to always cite reviewed excerpts even when stating that information is missing. I then validated the fix across multiple runs.

<!-- ── Stretch features ─────────────────────────────────────────────────────
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
| 1. Retrieved chunk contains the answer | 4 of 5 | 4/5 | 4/5 | 4/5 | MET |
| 2. Every answer names a source | 5 of 5 | 5/5 | 4/5 | 5/5 | MISSED |
| 3. Gate stops out-of-corpus questions | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 4. Every chunk contains question and answer | 5 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 5. At least one answer gives an alternative | 1 of 5 | 2/5 | 2/5 | 2/5 | MET |

### Output for each criterion

#### Criterion 1: Retrieved chunk contains the answer
- **Question:** The best secret study spot?
- **Produced by:** `store.py::search` (source: `thread_study_spots.txt`, distance: 0.5360)
```text
THREAD: Best study spots that aren't the library?

--- reply 2 (19 votes) ---
The science building has open lounges on floors 2 through 5 that are unlocked and almost always empty.
```

#### Criterion 2: Every answer names a source
- **Question:** Any strategies to save money on textbooks (Run 1)
- **Produced by:** `generate.py::answer_from_chunks`
```text
You can check the numbering of your textbook against the library reserve copy for free, as the library reserve copy is always the current edition (thread_textbook_editions.txt). Additionally, an emergency fund for textbooks exists and is not means-tested beyond a short form (thread_first_gen.txt).
```
- *Miss on Run 2 (Question: What do I do if I get sick the day of an exam?):*
```text
I do not have enough information to answer your question from the provided documents.
```

#### Criterion 3: Gate stops out-of-corpus questions
- **Question:** What is the capital of Mongolia?
- **Produced by:** `gate.py::check` / `run_eval.py::check_out_of_scope` (Best distance: 0.893 vs cutoff 0.65)
```text
refused (best distance 0.893)
I don't have enough information about that.
```

#### Criterion 4: Every chunk contains question and answer
- **Produced by:** `chunker.py::split_documents` (source: `thread_first_gen.txt#1`)
```text
THREAD: Anything specific for first-generation students?

--- reply 2 (41 votes) ---
The thing I'd say: the unwritten rules are the hard part, not the coursework. Ask about the unwritten rules explicitly. People are happy to explain them and nobody volunteers them.
```

#### Criterion 5: At least one answer gives an alternative
- **Question:** The best secret study spot? (Run 1)
- **Produced by:** `generate.py::answer_from_chunks`
```text
Based on the provided documents, there is no mention of a "secret" study spot. However, the thread about study spots mentions Ridgeway Café before 10am, open lounges in the science building, and the library's third floor or group study rooms (thread_study_spots.txt).
```

## Verdicts

<!-- MET or MISSED for each of the five, against the target you wrote last
     unit — not a new one. Plus a sentence on how you decided. That sentence
     matters most where it was close.

     If your target said 4 of 5 and your runs came out 4, 3, 4, that's a MISS.
     The target has to hold, not show up occasionally.

     Milestone 2. -->

| # | Criterion | Verdict | How I decided |
|---|---|---|---|
| 1 | Retrieved chunk contains the answer | MET | Across all three runs, 4 of 5 questions consistently had the answer in the top retrieved chunks (Question 4 was the only miss, retrieving late-work threads instead of exam policy). |
| 2 | Every answer names a source | MISSED | Target was 5 of 5; while Runs 1 and 3 achieved 5/5, Run 2 scored 4/5 because Question 4 produced a fallback refusal statement without citing any source filename. |
| 3 | Gate stops out-of-corpus questions | MET | The relevance gate refused all 5 out-of-scope questions (best distance was 0.807 against the 0.65 threshold), meeting the 4 of 5 target across all runs. |
| 4 | Every chunk contains question and answer | MET | `chunker.py::split_documents` prepends the `THREAD:` title to every reply block, ensuring 100% of chunks contain both the question and one answer. |
| 5 | At least one answer gives an alternative | MET | In all three runs, at least 2 questions (textbook savings and study spots) provided alternative or secondary answers, exceeding the target of at least 1 of 5. |
## Diagnoses

### Criterion 2: Every answer names a source (Missed on Run 2)

- **Failed question:** *"What do I do if I get sick the day of an exam?"*
- **Stage:** **Generation** (triggered by **Loading** and **Retrieval**)

**What happened:**
1. **Loading:** The documents don't actually have an answer about missing an exam. The closest thread is `thread_late_work.txt`, which only talks about handing in late homework.
2. **Retrieval & Gate:** Retrieval matched on words like "sick" and "illness" and returned `thread_late_work.txt` at distance 0.606. Because my cutoff was set to 0.65, this slipped past the gate when it should have been stopped.
3. **Generation:** In Run 2, the Gemini model saw that the chunks didn't answer the question and followed the prompt rule: *"If the documents don't cover the question, say you don't have enough information."* It replied *"I do not have enough information to answer your question from the provided documents."* Because it didn't use any file, it didn't name a file, leaving Run 2 at 4 of 5.

**Pattern:**
This was the only failure across all five questions. When a question has no answer in the documents but passes the gate anyway, the model is inconsistent: in Runs 1 and 3 it tried to answer using the late work thread and named the file, but in Run 2 it refused and named no file.

## The Improvement

**What I changed:**
In `generate.py`, I updated `GROUNDING_INSTRUCTION` so that Rule 3 explicitly instructs the model to always name the document(s) provided in the excerpts, even when stating that there is not enough information to answer.

**Why I picked it:**
My diagnosis showed that Criterion 2 missed on Run 2 because the model followed the prompt rule to refuse an uncovered question, but omitted the source filename because it did not use the document to answer. 

*(Note on test questions: Although my Question 4 turned out to be an edge case not explicitly answerable by the corpus, I kept the test questions identical rather than swapping it out. Changing questions after seeing the results would have invalidated the before-and-after comparison.)*

### Run Log — After

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 | 4/5 | 4/5 | 4/5 | MET |
| 2. Every answer names a source | 5 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 3. Gate stops out-of-corpus questions | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 4. Every chunk contains question and answer | 5 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 5. At least one answer gives an alternative | 1 of 5 | 1/5 | 1/5 | 1/5 | MET |

**Did it help?**
Yes. In the after run, Question 4 cited `thread_late_work.txt` across all three runs while still properly stating that the documents did not have enough information about exams. Criterion 2 improved from 4/5 on Run 2 to 5/5 across all three runs, moving from MISSED to MET, while all other criteria remained MET.

## What's Still Broken

While all five criteria met their targets in the after run, the pipeline still has an underlying weakness: Question 4 (*"What do I do if I get sick the day of an exam?"*) is not actually answerable from the corpus. Because the relevance cutoff is 0.65, Question 4 slips past the gate at distance 0.606 and retrieves `thread_late_work.txt`. Our updated prompt ensures the model cites that thread while stating it lacks exam details, but the system still cannot give the user a real exam answer.

To fix this properly, I would either add an explicit exam policy thread to the corpus or tighten the gate cutoff to 0.60 so the gate cleanly stops the question upfront. I stopped here because the assignment limits us to one measured fix, and tightening the prompt successfully resolved the citation inconsistency across all runs.

## What I'd Do Differently

1. **Criterion 2 ("Every answer names a source"):** In the next unit, I would reword this to: *"Every answer to an in-corpus question names a source"* or *"Every answer that provides advice names a source."* Demanding a source citation even when the system legitimately refuses an unanswerable question created a conflict between the refusal instruction and the citation requirement.
2. **Test Question Selection:** Before locking in test questions, I would search the documents directly to verify that every question has a clear, direct answer in the corpus, rather than assuming a related topic (late homework) covers exam policy.
