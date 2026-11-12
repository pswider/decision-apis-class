<p align="center">
  <img src="images/banner.svg" alt="Decision APIs in practice. A 45 to 50 minute class on Jev, Microsoft-Decision-1 and Gemini Decision 1, and what happened when one of them scored a year of Medicare cost report coding." width="100%">
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Length-45%20to%2050%20min-A76F3E?style=flat-square&labelColor=191919" alt="Length: 45 to 50 minutes">
  <img src="https://img.shields.io/badge/Level-Technical-A76F3E?style=flat-square&labelColor=191919" alt="Level: technical">
  <img src="https://img.shields.io/badge/APIs%20compared-3-A76F3E?style=flat-square&labelColor=191919" alt="APIs compared: 3">
  <img src="https://img.shields.io/badge/Code%20links-15-A76F3E?style=flat-square&labelColor=191919" alt="Code links: 15">
  <img src="https://img.shields.io/badge/Facts%20checked-November%2011%2C%202026-A76F3E?style=flat-square&labelColor=191919" alt="Facts checked November 11, 2026">
</p>

<p align="center">
  <a href="#1-run-of-show"><b>Run of show</b></a> &nbsp;·&nbsp;
  <a href="#2-what-a-decision-api-is"><b>Concepts</b></a> &nbsp;·&nbsp;
  <a href="#3-three-decision-apis-side-by-side"><b>The three APIs</b></a> &nbsp;·&nbsp;
  <a href="#6-code-tour-the-links"><b>Code tour</b></a> &nbsp;·&nbsp;
  <a href="#7-what-happened-on-real-data"><b>Results</b></a> &nbsp;·&nbsp;
  <a href="#8-what-went-wrong-and-what-it-taught"><b>Lessons</b></a> &nbsp;·&nbsp;
  <a href="#11-sources"><b>Sources</b></a>
</p>

<br>

What decision APIs are, how the three that matter right now compare, and what
happened when one of them was pointed at a real problem: checking how an
academic health system codes physician time for its Medicare cost report.

The real-world example is **Provider Activity Studio**, a Next.js and SQL
Server application, and its **Coding Review** screen, which uses TypeSafe
AI's **Jev** to score how sure a model is that each line of physician time was
coded to the right regulatory class.

> [!NOTE]
> **About the links.** The code links point at a private repository, pinned to
> one commit. They work for people who have been given access and show a 404
> for everyone else. This page carries enough on its own to follow along
> without them.

> [!IMPORTANT]
> **About the data.** Every screenshot shows invented lines. The health system
> is not named. Real results appear only as counts, and they are the model's
> output, not findings: no person has reviewed them yet.

<p align="center">
  <img src="images/01-coding-review-overview.png" alt="The Coding Review screen on invented lines: summary numbers, a Try it box, and a review queue sorted by code certainty" width="100%">
</p>

<details>
<summary><b>Contents</b></summary>

1. [Run of show](#1-run-of-show)
2. [What a decision API is](#2-what-a-decision-api-is)
3. [Three decision APIs, side by side](#3-three-decision-apis-side-by-side)
4. [The real problem](#4-the-real-problem)
5. [How the example works](#5-how-the-example-works)
6. [Code tour: the links](#6-code-tour-the-links)
7. [What happened on real data](#7-what-happened-on-real-data)
8. [What went wrong, and what it taught](#8-what-went-wrong-and-what-it-taught)
9. [Questions for the room](#9-questions-for-the-room)
10. [Running the demo](#10-running-the-demo)
11. [Sources](#11-sources)

</details>

| For the presenter | |
| --- | --- |
| [Speaker notes](docs/speaker-notes.md) | Minute by minute talking points, and answers to the questions you will get |
| [Examples](examples/) | A real Jev request and response, and the same question in Microsoft's and Google's shapes |
| [Style guide](docs/style-guide.md) | The colors and type used here, for slides that match |

---

## 1. Run of show

| Time | Min | Segment | What is on screen |
| --- | --- | --- | --- |
| 0:00 | 4 | **Hook.** Three phrases, three answers, about 100 ms each | [Try it](#the-hook-three-phrases) screenshots, or the live page |
| 0:04 | 7 | **What a decision API is.** Decide, not generate | [Section 2](#2-what-a-decision-api-is) |
| 0:11 | 7 | **The landscape.** Jev, Microsoft-Decision-1, Gemini Decision 1 | [Section 3](#3-three-decision-apis-side-by-side) |
| 0:18 | 5 | **The real problem.** Medicare, physician time, and why text matters | [Section 4](#4-the-real-problem) |
| 0:23 | 12 | **Code tour.** Six links, from database row to score | [Section 6](#6-code-tour-the-links) |
| 0:35 | 5 | **Results.** 8,324 lines for three cents | [Section 7](#7-what-happened-on-real-data) |
| 0:40 | 6 | **What went wrong.** Four privacy catches and one wrong assumption | [Section 8](#8-what-went-wrong-and-what-it-taught) |
| 0:46 | 4 | **Discussion and questions** | [Section 9](#9-questions-for-the-room) |

Total 50 minutes. To land at 45, shorten the code tour to links 1, 2, 3 and 5.

### The hook: three phrases

Type a job description. The model answers which kind of physician time it is,
with a probability for every option. These are real calls to `jev-1.14.0`
with invented text.

**A clear administrative role.** 99 percent provider component, in 92 ms.

![Try it: Medical Director, Intensive Care Unit](images/05-try-it-clear-admin.png)

**A clear patient-care activity.** 98 percent patient care. If this line were
coded as administration, it would be flagged.

![Try it: Outpatient clinic sessions](images/06-try-it-patient-care.png)

**Text that says nothing.** "Special projects". The model puts 84 percent on
"the text does not say". Look closely at the two numbers on the left: the top
class is at 9 percent while the model's confidence is 82 percent. Those are
different things, and the class comes back to that in section 2.

![Try it: Special projects](images/07-try-it-unclear.png)

---

## 2. What a decision API is

A chat model **generates**: you send a prompt and get text back, then parse
it. A decision API **decides**: you send some state and a set of typed
questions, and you get typed answers with probabilities. It writes no text.

<p align="center">
  <img src="images/generate-vs-decide.svg" alt="Generate: prompt, model, text you must parse. Decide: state and typed questions, model, typed answers with probabilities." width="100%">
</p>

| | Generative call | Decision call |
| --- | --- | --- |
| You send | A prompt | State, plus named questions with a type each |
| You get | Text (maybe JSON you asked for) | A typed answer and a probability per option |
| Output tokens | Billed, and the slow part | None |
| Latency | Seconds | Tens to low hundreds of milliseconds |
| Parsing | Yours to get right | Nothing to parse |
| "How sure?" | Ask the model, and distrust the answer | A number you can threshold |
| Explanation | Yes | No. You get a decision, not a reason |

### The question types

All three vendors use the same three, with the same names.

| Type | Question it answers | What comes back |
| --- | --- | --- |
| `noul` | Is this true? | One probability of yes, 0 to 1 |
| `choice` | Which option is it? | The chosen option and a probability for every option |
| `score` | How much? | A level on a scale, and a probability for every level |

### Five ideas to leave with

1. **Probability is the product.** The answer matters less than the number
   beside it. A queue sorted by that number is the application.
2. **Confidence is not probability.** Jev's docs say `confidence` measures how
   concentrated the probabilities are, not the chance the answer is right.
   Threshold on the probability of the option your code will act on.
3. **Calibration is a claim until you measure it.** "A 90 percent prediction
   should be right nine times in ten" is a property of the vendor's test
   cases. On your data it is a hypothesis. The example below builds the
   measurement in: people record verdicts, and the screen shows how often
   low-certainty lines were really wrong.
4. **Always give an escape option.** "The text does not say" is a legitimate
   answer. Without it the model must pick a category for noise.
5. **Keep arithmetic in code.** These models read; they do not calculate.
   Numbers never go to the model in the example, and every sum is done
   locally.

### When not to reach for one

When you need the reasoning, when the answer is open-ended, or when the input
needs facts the model cannot see. A decision API gives you a fast, cheap,
rankable signal. It does not give you an argument you can show an auditor.

---

## 3. Three decision APIs, side by side

Checked against the vendors' own pages on November 11, 2026. Things move
weekly in this category; treat the table as a snapshot.

<p align="center">
  <img src="images/timeline.svg" alt="Jev, September 15, 2026. Microsoft-Decision-1, October 9, 2026. Gemini Decision 1, November 11, 2026." width="100%">
</p>

| | **Jev** | **Microsoft-Decision-1** | **Gemini Decision 1** |
| --- | --- | --- | --- |
| From | TypeSafe AI | Microsoft, in Microsoft Foundry | Google DeepMind, in the Gemini API |
| Announced | September 15, 2026 | October 9, 2026 | November 11, 2026 |
| Endpoint | `POST api.typesafe.ai/v1/systemone` | `POST <foundry-endpoint>/providers/microsoft/v1/systemone` | `POST generativelanguage.googleapis.com/v1beta/decisions` |
| Auth | Bearer key | Entra ID token or `api-key` | `x-goog-api-key` |
| Request fields | `model`, `state`, `questions` | `model`, `state`, `questions` | `model`, `input`, `questions` |
| Question types | `noul`, `choice`, `score`, `box` | `noul`, `choice`, `score` | `noul`, `choice`, `score` |
| Input | Text, JSON, images | Text, JSON | Text, images, audio, video, PDF |
| Context | 64k per request, 32k per question (262k opt-in) | Not stated | 262,144 tokens |
| Questions per request | Not stated | Not stated | 1 to 1,000 |
| Price, per 1M input tokens | $0.042 | $0.042 | $0.10 |
| Output tokens | Free | Free | Free |
| Calibration claim | Trained for calibrated decisions; no metric published in text | "a 90% prediction should be right about nine times out of 10 on representative cases" | "lowest calibration error" on Google's own suite |
| Vendor's own caution | Confidence is not correctness | "Calibration is strongest on familiar task types" | Validate on your own data |

Things worth saying out loud:

- **The schema is converging.** Microsoft's endpoint path ends in
  `/v1/systemone` and takes `state` and `questions`, the same shape as Jev's.
  Google renamed `state` to `input` and kept the rest. Code written against
  one is a small adapter away from the others. Microsoft's post cites a
  leaderboard called JevBench; neither Microsoft page states a relationship
  with TypeSafe.
- **Microsoft says what it is built on.** The announcement says the model was
  post-trained from Qwen3.5-9B and will be rebased on Microsoft AI and OpenAI
  models. A small base model is a large part of why these calls are fast and
  cheap.
- **Google's bet is multimodal and wide.** Up to 1,000 independent questions
  over one shared input that can be video or a PDF.
- **Everyone bills input only**, and every vendor tells you to check
  thresholds on your own labelled data. Nobody publishes a calibration number
  you can copy into a design document.
- **Others are reported to be shipping.** Secondary sources describe an OpenAI
  Decisions API and several more. We did not confirm those against primary
  pages, so they are not in the table.

The minimal request for each is in [examples/](examples/). Only the Jev one
was run by us.

---

## 4. The real problem

A teaching hospital pays its physicians for more than seeing patients. A
department chair runs a department. A program director trains residents.
Medicare treats these differently:

| Kind of time | Who pays | Rule |
| --- | --- | --- |
| **Provider component**: administration, supervision of staff, committee and quality work, availability | Medicare Part A, through the hospital's cost report | 42 CFR 415.55 and 415.60 |
| **Professional**: care of individual patients | Medicare Part B, through claims | |
| **Teaching** residents and fellows | Graduate medical education payments | 42 CFR 413.75 and following |
| Research, outside work, leave | Not claimed | |

The hospital may claim only the provider component on the cost report
(Worksheet A-8-2 of form CMS-2552-10), the claim must rest on a documented
allocation of each physician's time, and the amount is capped by specialty
(the Reasonable Compensation Equivalent limit, 42 CFR 415.70). The thing
Medicare most wants to prevent is **paying twice for the same hours**: once
as administration on the cost report and again as patient care on a claim.

In practice each physician has a handful of lines each month:

```
A&S   0.100 FTE   "Medical Director, Sleep Laboratory"
GME   0.200 FTE   "Program Director, Internal Medicine Residency"
```

Someone typed the description. Someone chose the code. The physician signed
the month. Nobody ever reads all of them again. At this health system that is
about 7,500 claimed lines a year.

**The question for a decision API:** given only the words in the description,
how sure are you that the code is right?

> [!NOTE]
> This is not legal or compliance advice. The regulation is summarized to make
> the example understandable.

---

## 5. How the example works

<p align="center">
  <img src="images/architecture.svg" alt="A line is read from a de-identified database, two text fields pass through redaction, they are posted with three typed questions to the decision API, the probabilities become a score, and the score orders a review queue where a person records a verdict." width="100%">
</p>

### The question we ask

One `choice` question with ten options, taken from the categories of a
physician time study, plus an escape:

| Option sent to the model | Class |
| --- | --- |
| Administration or medical direction of a hospital department, unit, service or program | Provider |
| Supervising or training nurses, technicians or other hospital staff | Provider |
| Committee work, utilization review, quality assurance or quality improvement | Provider |
| On-call or emergency availability paid by the hospital, not tied to a specific patient | Provider |
| Teaching or supervising residents or fellows in an accredited training program | GME |
| Medical or surgical care of individual patients | Professional |
| Research, whether sponsored or internally funded | Other funded |
| Work for another organization, fundraising, or other activity the hospital does not claim | Other funded |
| Leave, sabbatical or other paid time away | Leave |
| The text does not say | none |

Two more questions ride on the same call: a `noul` ("is this time spent on
duties that benefit patients generally?") and a `choice` for the specialty
group. Three questions, one request, about 850 input tokens.

The option order is **shuffled per payload** from a seed, because the vendor
says the model leans toward the first option.

### The score

Several options map to one class, so:

> **Code certainty** = the sum of the probabilities of every option whose
> class is the class the line is already coded as.

It answers "how sure is the model that the existing code is right". The queue
sorts on it, lowest first.

<p align="center">
  <img src="images/score-bands.svg" alt="Code certainty from 0 to 100 percent. At or below 15 percent: likely miscoded. Between 15 and 85 percent: worth a look. At or above 85 percent: confirmed. Unclear text is checked first, whatever the certainty." width="100%">
</p>

| Band | Rule (defaults: high 0.85, mid 0.60) | What a reviewer does |
| --- | --- | --- |
| Likely miscoded | Code certainty at or below 0.15 | Open first |
| Unclear text | The model chose "does not say", or its top class is under 0.60 | A documentation problem in its own right |
| Confirmed | Code certainty at or above 0.85 | Skip |
| Worth a look | Anything else | Second tier |

Unclear text is checked first: a line whose text says nothing is never called
miscoded or confirmed.

A likely-miscoded line also gets a **direction**. *Overclaim* means the line is
claimed (provider or teaching) and the text reads as something not claimable.
That is the one Medicare cares about.

### What a reviewer sees

The queue, lowest certainty first:

![Review queue](images/02-review-queue.png)

Open a line and the screen shows three things side by side: exactly what was
sent, what stayed on the machine, and how the model read it.

![What was sent and what was kept](images/03-line-detail-sent-vs-kept.png)

Filter to overclaims and sort by hours, and you have a worklist:

![Overclaims only](images/04-overclaim-filter.png)

---

## 6. Code tour: the links

Every link is pinned to commit
[`13cc75a`](https://github.com/realactivity/provider-activity-studio/tree/13cc75a15459143c03a92f6b218b77b42e10ae16),
so the line numbers will not drift. The repository is private; see the note at
the top.

### The six to click in class

**1. The REST call.**
[`web/lib/decisions/client.ts`, lines 55 to 71](https://github.com/realactivity/provider-activity-studio/blob/13cc75a15459143c03a92f6b218b77b42e10ae16/web/lib/decisions/client.ts#L55-L71)

The whole integration is one `fetch`: `POST {baseUrl}/v1/systemone`, a bearer
key, and a JSON body of `{ model, state, questions }`. No SDK. **Why it
matters:** this is all a decision API asks of you. There is no prompt
template, no streaming, no output parsing. Scroll up to
[lines 33 to 46](https://github.com/realactivity/provider-activity-studio/blob/13cc75a15459143c03a92f6b218b77b42e10ae16/web/lib/decisions/client.ts#L33-L46)
for the strict validation of what comes back, and down to
[lines 73 to 109](https://github.com/realactivity/provider-activity-studio/blob/13cc75a15459143c03a92f6b218b77b42e10ae16/web/lib/decisions/client.ts#L73-L109)
for the timeout, two retries on 429 and 529, and a circuit breaker after five
failures.

**2. The questions.**
[`web/lib/decisions/contract.ts`, lines 25 to 58](https://github.com/realactivity/provider-activity-studio/blob/13cc75a15459143c03a92f6b218b77b42e10ae16/web/lib/decisions/contract.ts#L25-L58)
and
[lines 91 to 102](https://github.com/realactivity/provider-activity-studio/blob/13cc75a15459143c03a92f6b218b77b42e10ae16/web/lib/decisions/contract.ts#L91-L102)

The ten options and their classes, the three instructions, and
`buildQuestions`, which shuffles the option order from a seed. **Why it
matters:** with a decision API the "prompt engineering" is the option list.
One condition per option, plain words, an escape option, and a defence
against position bias. The domain knowledge of the whole feature lives in
these thirty lines.

**3. The score.**
[`web/lib/decisions/contract.ts`, lines 186 to 237](https://github.com/realactivity/provider-activity-studio/blob/13cc75a15459143c03a92f6b218b77b42e10ae16/web/lib/decisions/contract.ts#L186-L237)

`derive()` turns the model's probabilities into code certainty, a band and a
direction. **Why it matters:** this is where a vendor's number becomes your
product's number. It sums probabilities by class, ignores the vendor's
`confidence` for the decision, and applies thresholds you own. It is a pure
function with tests, so the scoring rule can be argued about and changed
without touching the network.

**4. The pipeline.**
[`web/lib/decisions/score.ts`, lines 42 to 115](https://github.com/realactivity/provider-activity-studio/blob/13cc75a15459143c03a92f6b218b77b42e10ae16/web/lib/decisions/score.ts#L42-L115)

Four steps: redact every line, collapse identical text to one question,
answer from the cache where possible, ask the model for the rest with bounded
concurrency. **Why it matters:** 7,472 lines needed only 917 distinct
payloads, because a role repeats every month. Deduplication and a
content-addressed cache
([`payloadHash`, lines 115 to 121](https://github.com/realactivity/provider-activity-studio/blob/13cc75a15459143c03a92f6b218b77b42e10ae16/web/lib/decisions/contract.ts#L115-L121))
cut the calls by a factor of eight and make a rerun free.

**5. The only way out.**
[`web/lib/decisions/redact.ts`, lines 11 to 17](https://github.com/realactivity/provider-activity-studio/blob/13cc75a15459143c03a92f6b218b77b42e10ae16/web/lib/decisions/redact.ts#L11-L17),
[lines 38 to 59](https://github.com/realactivity/provider-activity-studio/blob/13cc75a15459143c03a92f6b218b77b42e10ae16/web/lib/decisions/redact.ts#L38-L59)
and
[lines 76 to 111](https://github.com/realactivity/provider-activity-studio/blob/13cc75a15459143c03a92f6b218b77b42e10ae16/web/lib/decisions/redact.ts#L76-L111)

An allowlist of two text fields per source, then a scrub that removes roster
names, emails, account names, decimals, percentages, long digit runs and
letter-digit codes. Nothing reaches the API except through `toState`. **Why
it matters:** half of these patterns did not exist on the first day. Each one
was added after a dry run found something in real text. Section 8 tells that
story; this file is the result.

**6. From a database row to a case.**
[`web/lib/decisions/cases.ts`, line 118](https://github.com/realactivity/provider-activity-studio/blob/13cc75a15459143c03a92f6b218b77b42e10ae16/web/lib/decisions/cases.ts#L118)
and
[lines 162 to 198](https://github.com/realactivity/provider-activity-studio/blob/13cc75a15459143c03a92f6b218b77b42e10ae16/web/lib/decisions/cases.ts#L162-L198)

The read-only query over the finance lines and `toFinanceCase`, which decides
what a line is coded as, what text may be sent (`source`) and what stays
(`local`). **Why it matters:** the split between `source` and `local` is the
privacy design, expressed as a type. It is also where the lines that are
never sent get counted, with a reason for each.

### More, if there is time

<details>
<summary><b>Nine more links</b>: the guard, the cache, the schema, the tests, the mock</summary>
<br>

| Link | What it is | Why it is worth a look |
| --- | --- | --- |
| [`config.ts`, lines 46 to 91](https://github.com/realactivity/provider-activity-studio/blob/13cc75a15459143c03a92f6b218b77b42e10ae16/web/lib/decisions/config.ts#L46-L91) | The feature flag and the database guard | The feature refuses to run against a production database whatever the flag says. A guard in code beats a rule in a document. |
| [`run.ts`, lines 25 to 66](https://github.com/realactivity/provider-activity-studio/blob/13cc75a15459143c03a92f6b218b77b42e10ae16/web/lib/decisions/run.ts#L25-L66) | One run: sample, score, store | Every source of lines goes through the same path, so they cannot drift apart. |
| [`store.ts`, lines 113 to 130](https://github.com/realactivity/provider-activity-studio/blob/13cc75a15459143c03a92f6b218b77b42e10ae16/web/lib/decisions/store.ts#L113-L130) | The answer cache lookup | Saved answers by payload hash. This is what makes "replay" possible: present with no network and no key. |
| [`create-jev-schema.sql`](https://github.com/realactivity/provider-activity-studio/blob/13cc75a15459143c03a92f6b218b77b42e10ae16/scripts/sql/create-jev-schema.sql#L19-L76) | Three tables: runs, predictions, verdicts | The verdict table is the calibration experiment. |
| [`redact.test.ts`, lines 104 to 141](https://github.com/realactivity/provider-activity-studio/blob/13cc75a15459143c03a92f6b218b77b42e10ae16/web/lib/decisions/redact.test.ts#L104-L141) | Tests for the scrub | Each test is a thing that was found in real text. Read them as an incident log. |
| [`mock.ts`, line 49](https://github.com/realactivity/provider-activity-studio/blob/13cc75a15459143c03a92f6b218b77b42e10ae16/web/lib/decisions/mock.ts#L49) | A keyword engine with the same answer shape | The whole feature was built and tested before a key existed, and every dry run uses it. |
| [`fixtures.ts`](https://github.com/realactivity/provider-activity-studio/blob/13cc75a15459143c03a92f6b218b77b42e10ae16/web/lib/decisions/fixtures.ts#L14-L61) | The forty invented lines in the screenshots | Scored through the real pipeline, so the demo data can never drift from the contract. |
| [`app/api/decisions/run/route.ts`](https://github.com/realactivity/provider-activity-studio/blob/13cc75a15459143c03a92f6b218b77b42e10ae16/web/app/api/decisions/run/route.ts) | The HTTP route that starts a run | Where funded and grant lines are refused. |
| [`docs/JEV_DEV_DEMO.md`](https://github.com/realactivity/provider-activity-studio/blob/13cc75a15459143c03a92f6b218b77b42e10ae16/docs/JEV_DEV_DEMO.md) | The build log | Every step, every number, every mistake, in order. |

</details>

---

## 7. What happened on real data

One fiscal year of claimed lines (administration and teaching) for physicians
at an academic health system, on a de-identified copy of the database, scored
by `jev-1.14.0` on November 10, 2026.

<p align="center">
  <img src="images/stats.svg" alt="7,472 lines scored, 917 distinct texts, 771 model calls, 64 milliseconds median latency, 3 cents total cost." width="100%">
</p>

| Scope | Lines | Physicians | Distinct texts | Model calls | Cost |
| --- | --- | --- | --- | --- | --- |
| The whole system | 7,472 | 484 | 917 | 771 | $0.0274 |
| Hospital B on its own | 739 | 58 | 91 | 89 | $0.0032 |
| Hospital C on its own | 113 | 12 | 15 | 15 | $0.0005 |

The whole-system run includes hospitals B and C, so the rows do not add up.
A line is one role in one month.

- **Speed.** Median 64 ms a call. The first 50-line run took 2.8 seconds end
  to end.
- **Cost.** About three cents for everything, including the test calls.
- **Failures.** None. No call failed and no line was left unscored.
- **The request and response shapes** taken from the vendor's documentation
  worked on the first call, with strict validation switched on.

How the model read the whole-system lines:

<p align="center">
  <img src="images/results.svg" alt="Administration, 5,628 lines: 51 percent confirmed, 15 percent worth a look, 26 percent unclear text, 8 percent likely miscoded. Teaching, 1,844 lines: 19 percent confirmed, 25 percent worth a look, 50 percent unclear text, 7 percent likely miscoded." width="100%">
</p>

<details>
<summary><b>The same numbers as a table</b></summary>
<br>

| Coded as | Lines | Confirmed | Worth a look | Unclear text | Likely miscoded |
| --- | --- | --- | --- | --- | --- |
| Administration (provider component) | 5,628 | 2,892 | 816 | 1,486 | 434 |
| Teaching | 1,844 | 347 | 459 | 914 | 124 |

</details>

Three things to draw out:

1. **Half of the administration lines are confirmed outright.** A reviewer can
   skip 2,892 lines.
2. **405 administration lines read as overclaims**, and they come from only
   **35 distinct role texts**. Thirty-five human judgments would settle most
   of that number, one way or the other. That is the pitch for the whole
   approach: the model does not find errors, it tells a person which
   thirty-five things to read out of seven thousand.
3. **About a third of the lines are "unclear text".** The role is two or
   three words, or an acronym. Nobody asked for this finding, and it may be
   the most useful one: the documentation behind a third of the claimed time
   would not tell an outsider what the time was for.

> [!WARNING]
> **Read this before quoting a number.** These are model outputs on short
> text. At the time of writing no person had recorded a single verdict, so
> the calibration of these scores on this data is unknown. "Likely miscoded"
> means "read this first", not "this is wrong".

---

## 8. What went wrong, and what it taught

The most useful ten minutes of the project were the dry runs. Every one of
these was caught with a keyword mock in place of the model, before anything
left the machine.

| What we found | How | What changed |
| --- | --- | --- |
| **People type numbers into free text.** Role descriptions held effort fractions, percentages and cost centre numbers. The rule was "never send effort numbers", and the allowlist only checked which column was sent, not what people had written in it. | A check of stored payloads for any digit | The scrub now removes decimals, percentages, runs of four or more digits, and codes that join letters to digits. Short whole numbers that name a ward stay. |
| **Some text cannot be made safe.** Lines for grants and funds carried grant numbers, donor fund names and an investigator's name. A name filter built from the staff roster cannot recognize a person who is not on the roster. | A wider dry run that included those lines | Those lines are never sent. The route refuses them. Only administration and teaching lines are scored. |
| **The anonymizer left a fingerprint.** It had replaced a surname inside an institution's name with a stand-in, and the stand-in was in nobody's roster row, so the filter let it through. | Matching every payload against every stand-in name as well as every real one | The name filter now also strips every stand-in name the anonymizer ever issued. |
| **The first check passed and proved nothing.** A 50-line review of payloads from the first data source found no problems. It could not have: that source's text had none of these patterns. | The second source failed the same check | Before sending, every line of the actual run is dry-run and checked, not a sample from somewhere similar. |
| **The text was not where the schema said.** The view built for the report carries descriptions only for teaching lines. Administration lines, the ones the regulation is about, say only a placeholder there. The first scope judged nothing but teaching. | Every sampled line turned out to be a teaching line | A second scope reads the finance lines, where the administration text actually lives. |
| **A data assumption was simply false.** A second use case needed monthly productivity (RVU) figures. The column existed and had been empty since 2016. | A profile query, before building | That use case was not built. |

The lessons that generalize:

- **An allowlist of columns is not an allowlist of content.** Free text
  contains whatever people typed.
- **Dry-run the exact population you are about to send**, with an engine that
  cannot leak.
- **Check outputs you can count.** "No payload contains a digit run of four
  or more" is a test. "We removed identifiers" is a hope.
- **Decide what you will never send**, and enforce it in the route, not in a
  comment.
- **Profile the data before designing the feature.** Two of the six rows
  above are about data that was not what everyone believed.
- **Build the calibration experiment into the product.** The verdict buttons
  are not a nicety. Without them the scores stay a vendor claim.

> [!TIP]
> If the room remembers one line: **an allowlist of columns is not an
> allowlist of content.**

---

## 9. Questions for the room

1. The model put 9 percent on its top class and reported 82 percent
   confidence ("Special projects"). Which number would you threshold on, and
   why?
2. Thirty-five role texts account for 405 flagged lines. What does that say
   about where human review time should go? What could go wrong with that
   shortcut?
3. A third of the lines are "unclear text". Is that a failure of the model,
   of the data, or a finding? Who in the organization should hear it?
4. The three APIs share a schema. Would you write to one vendor or put an
   adapter in front? What would you need to re-validate when you switch?
5. A decision API gives no rationale. What would you show an auditor who
   asks why a line was flagged?
6. Where in your own work is there a pile of short text that someone coded by
   hand and nobody rereads?
7. The scrub removes "20%" from text before the model sees it. Could that
   change the model's answer? How would you find out?

---

## 10. Running the demo

For people with access to the private repository. No database and no key are
needed for this mode: it serves the forty invented lines.

```bash
cd web
npm ci
DECISIONS_ENABLED=on DECISIONS_FIXTURES=synthetic AUTH_MODE=development \
  DEV_USER='DEV\analyst' AUTH_DEFAULT_ROLE=operator npm run dev
# open http://localhost:3007/decisions
```

Add a TypeSafe key as `TYPESAFE_API_KEY` in `web/.env.local` and "Try it"
calls the real model with whatever you type. Type invented text only.

Three ways to present, in order of how much can go wrong on the day:

| Mode | Needs | Good for |
| --- | --- | --- |
| Snapshot file | Nothing | The safest. A saved run, read-only. |
| Replay | The database | Real runs from saved answers. No network, no key. |
| Live | The database, a key, the network | "Try it" with phrases from the audience. |

---

## 11. Sources

**Jev (TypeSafe AI)**

- Quickstart: endpoint, request and response. <https://docs.typesafe.ai/introduction/quickstart.md>
- Models: versions, price, context limits. <https://docs.typesafe.ai/models.md>
- Confidence versus probability. <https://docs.typesafe.ai/confidence.md>
- Errors and request limits. <https://docs.typesafe.ai/api/errors.md>
- Data terms. <https://docs.typesafe.ai/legal.md>
- Launch post. <https://typesafe.ai/blog/introducing-system-one-models-and-jev>

**Microsoft-Decision-1**

- Announcement, October 9, 2026: base model, benchmarks, price, calibration claim. <https://commandline.microsoft.com/microsoft-decision-1-model-foundry/>
- Microsoft Learn how-to: endpoint, authentication, question types, limitations. <https://learn.microsoft.com/en-us/azure/foundry/foundry-models/how-to/use-foundry-models-microsoft-decision>
- Foundry blog: availability and pricing table. <https://techcommunity.microsoft.com/blog/azure-ai-foundry-blog/introducing-microsoft-decision-1-in-microsoft-foundry-for-decision-and-classific/4562742>

**Gemini Decisions API (Google DeepMind)**

- Announcement, November 11, 2026: model, price, claims. <https://blog.google/innovation-and-ai/technology/developers-tools/gemini-decisions-api/>
- Documentation: endpoint, fields, limits. <https://aistudio.google.com/docs/decisions-api>
- A worked walkthrough on dev.to. <https://dev.to/philschmid/fast-multimodal-control-flow-with-the-gemini-decisions-api-533b>

**Medicare rules behind the example**

- 42 CFR 415.55 and 415.60, allowable costs and allocation of physician compensation. <https://www.ecfr.gov/current/title-42/chapter-IV/subchapter-B/part-415/subpart-B>
- 42 CFR 415.70, limits on compensation for physician services in providers. <https://www.ecfr.gov/current/title-42/section-415.70>
- 42 CFR 413.75 and following, direct graduate medical education payments. <https://www.ecfr.gov/current/title-42/section-413.75>
- Provider Reimbursement Manual, Part 2, Chapter 40 (form CMS-2552-10, Worksheet A-8-2). <https://www.cms.gov/regulations-and-guidance/guidance/manuals/paper-based-manuals-items/cms021935>

**How the facts on this page were checked**

- The vendor pages above were read on November 11, 2026, and the comparison
  table was checked against them a second time, independently.
- Not confirmed against a primary page, and therefore left out of the table:
  reports of an OpenAI Decisions API and of decision models from other
  vendors.
- Everything in sections 5 to 8 comes from the project's own code, tests and
  build log.
- The Jev request and response in [examples/](examples/) are from a real
  call. The Microsoft and Google samples are shapes from their documentation
  and were not run.

---

<p align="center">
  <sub>
    Independent class materials. Not affiliated with, sponsored by or endorsed by Microsoft, Google or TypeSafe AI.<br>
    Product names belong to their owners. The colors and type follow the public Microsoft AI site as a courtesy to the venue; no logos are used.<br>
    <a href="docs/style-guide.md">Style guide</a> &nbsp;·&nbsp; <a href="docs/speaker-notes.md">Speaker notes</a> &nbsp;·&nbsp; <a href="examples/">Examples</a>
  </sub>
</p>
