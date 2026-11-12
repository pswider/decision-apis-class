# Speaker notes

[Back to the class](../README.md) · [Examples](../examples/) · [Style guide](style-guide.md)

Minute by minute for a 50 minute slot. Everything on screen is the
[README](../README.md); scroll it, do not switch windows more than you have
to. Times are when each segment starts.

## Before the room fills

- Open the README in one tab and the six code links in six more, in order.
  Sign in to GitHub first, or the links show a 404.
- If you will demo live, start the app in synthetic mode (README section 10)
  and load `/decisions` once so the first click is not a cold start.
- Decide now whether "Try it" will be live. If the network is doubtful, use
  the three screenshots. They are real calls.
- Say early, once: the health system is not named, the screenshots are
  invented lines, and the result counts are model output nobody has reviewed.

## 0:00 Hook (4 min)

Goal: the room sees a decision API answer before hearing a definition.

- "I'm going to type a job description and ask a model one question: what
  kind of physician time is this?"
- Phrase one, "Medical Director, Intensive Care Unit". 99 percent
  administration, 92 milliseconds. Point at the bar chart: every option got a
  number, not just the winner.
- Phrase two, "Outpatient clinic sessions, general cardiology". 98 percent
  patient care. "If a hospital had coded this as administration and billed
  Medicare for it, this is the line an auditor wants."
- Phrase three, "Special projects". Pause here. "It did not guess. 84 percent
  on 'the text does not say'." Then point at 9 percent and 82 percent and
  leave it hanging: "two different numbers. We'll come back to why."
- Land it: "No prompt. No parsing. About a hundred milliseconds, and it cost
  four hundred-thousandths of a dollar."

If live: ask the room for a job title. Take one that is ambiguous on purpose.

## 0:04 What a decision API is (7 min)

Goal: three ideas stick. Decide, not generate. Probability is the product.
Confidence is not probability.

- Walk the two-column table. Stress the "Explanation" row: you give up the
  reason. That is the trade.
- The three question types. Say the names out loud; they are identical across
  vendors. `noul` is a yes or no with a probability.
- Return to "Special projects". Confidence 82 percent means "I am decisive
  that this is unclear". It does not mean "82 percent likely to be right
  about research". Jev's own docs say to threshold on the probability of the
  option you will act on.
- Calibration: "Every vendor says a 90 should be right nine times in ten. On
  whose data? Hold that thought until we get to verdicts."
- One sentence on what not to use it for: anything where you need the
  argument, not the answer.

## 0:11 The landscape (7 min)

Goal: the room understands this is a category that formed in eight weeks,
and that the schemas are converging.

- Dates first: September 15, October 9, November 11. "The Google one is
  hours old."
- Read across the endpoint row. Microsoft's path ends in `/v1/systemone`,
  the same as Jev's, with the same `state` and `questions`. Let people react.
  Be precise: neither Microsoft page states a relationship with TypeSafe;
  the announcement does cite a leaderboard called JevBench.
- In this room: Microsoft says in its own post that Decision-1 was
  post-trained from Qwen3.5-9B and will be rebased on MAI and OpenAI models.
  Quote it rather than paraphrase it.
- Price: $0.042 against $0.10 per million input tokens, output free
  everywhere. "At these prices the cost question is over. The question is
  whether the number means anything on your data."
- Google's differentiator: up to 1,000 questions over one input, and the
  input can be video.
- Do not get drawn on the vendors we did not verify. Say they are reported
  and that we did not confirm them.

## 0:18 The real problem (5 min)

Goal: a room of engineers understands why a sentence of text is worth money.

- "A hospital pays a physician to do four kinds of things. Medicare pays for
  them through three different doors, and one of them not at all."
- Use the four-row table. The line to remember: administration goes on the
  cost report, patient care goes on a claim, and the sin is doing both for
  the same hours.
- Show the two-line example. "Someone typed that description. Someone picked
  that code. The physician signed the month. Nobody reads it again. There are
  seven and a half thousand of these a year."
- The question: "Given only the words, how sure are you the code is right?"
- Say once: this is a simplification for the class, not compliance advice.

## 0:23 Code tour (12 min)

Goal: follow one line from a database row to a score. Two minutes a link.

1. **The REST call** (`client.ts`). "This is the entire integration." Read
   the three fields of the body aloud. Scroll up to the validation: it
   rejects any answer that is not exactly what was asked. Scroll down to
   retries and the breaker.
2. **The questions** (`contract.ts`, top). "This is the prompt engineering."
   Ten options, one condition each, and an escape. Then `buildQuestions`:
   ask why the order is shuffled. Answer: position bias, and the vendor says
   so.
3. **The score** (`contract.ts`, `derive`). The key line sums probabilities
   by class and reads off the one the line is coded as. "Notice we do not use
   the vendor's confidence to decide anything." Walk the four bands.
4. **The pipeline** (`score.ts`). Four numbered comments. Redact, dedupe,
   cache, ask. "7,472 lines became 917 questions."
5. **The only way out** (`redact.ts`). Start at the allowlist: two fields.
   Then the wall of regular expressions. "Half of these did not exist when
   we started. Hold that thought for five minutes."
6. **Row to case** (`cases.ts`). Point at `source` and `local` in the return
   value. "The privacy design is this type. One of those goes to the scrub.
   The other never leaves."

If short on time: 1, 2, 3, 5.

Show the line-detail screenshot here: "sent to the model" beside "kept here".
It is the code tour as a user sees it.

## 0:35 Results (5 min)

Goal: the scale and the cost land, and the caveat lands harder.

- "One fiscal year. 7,472 lines, 484 physicians. 771 calls. Under three
  cents. Sixty-four milliseconds a call."
- "Half the administration lines are confirmed. A reviewer can skip them."
- The number to dwell on: 405 lines read as overclaims, from 35 distinct
  role texts. "The model did not find 405 errors. It told a person which 35
  sentences to read."
- A third unclear. "Nobody asked for that finding."
- Then stop and say it plainly: "Nobody has checked any of these. Not one.
  Until someone does, this is a ranking, not a result."

## 0:40 What went wrong (6 min)

Goal: this is the part people will repeat to colleagues.

Tell it as a sequence, one sentence each, from the table:

1. "Our rule was: never send effort numbers. So we sent only the description
   column. People had typed effort numbers into the description."
2. "We widened the run to grant lines. Grant titles have investigators'
   names in them. Our name filter was built from the staff roster. Those
   people are not on the roster. We stopped sending those lines at all."
3. "The anonymizer had rewritten a surname inside the name of an
   institution. The replacement was in nobody's record, so the filter did
   not know it."
4. "Our first privacy check passed. It could not have failed: it checked
   text from a different source."
5. "And the feature was reading the wrong table. The view had descriptions
   only for teaching. The lines the regulation is about said a placeholder."

Then: "Every one of those was caught with a keyword mock standing in for the
model. Nothing left the building." Go back to link 5 and show the tests:
each test is one of these stories.

Close the segment on the six lessons. If you say only one: **an allowlist of
columns is not an allowlist of content.**

## 0:46 Discussion (4 min)

Pick two from README section 9. The two that reliably start a conversation:

- Question 1 (which number do you threshold on) checks that the confidence
  idea landed.
- Question 6 (where is your pile of hand-coded short text) sends people home
  with their own use case.

Last line: "Three vendors, one schema, a few cents. The hard part was never
the API. It was knowing what you are allowed to send, and proving the score
means something."

## Questions you will get

- **"Why not just use a chat model with JSON output?"** You can. You pay for
  output tokens, wait seconds, and get a self-reported confidence. The
  decision API gives a probability per option in one pass. For a ranking
  over thousands of lines, that is the difference between a batch job and a
  page load.
- **"Is it actually calibrated?"** Unknown on this data. The verdict buttons
  exist to find out. Until there are a few dozen verdicts per band, treat
  the score as an ordering.
- **"Is the health system's data going to the vendor?"** Two text fields per
  line, after scrubbing, from a de-identified copy. No names, ids or numbers.
  Grant and fund lines are never sent. The vendor's terms and data retention
  options are in the sources.
- **"Did it find fraud?"** No. It ranked lines for a person to read. Many
  flagged lines will turn out to be poorly worded, not wrongly coded.
- **"What is the relationship between Microsoft's model and Jev?"** The
  request schema and endpoint path match, and Microsoft's post cites
  JevBench. Neither Microsoft page states a relationship. Do not speculate
  past that.
- **"Could you swap vendors?"** The adapter is small: one field name and the
  auth header. The work is re-validating thresholds, because calibration is
  per model.
