# Sprint Assessment Guide
## Marketing Analytics — Student Reference

---

## What Is the Sprint?

The Sprint is the capstone assessment of the course. It is a structured case analysis that
brings together the course's analytical models. You will work as a group on a real business
problem (a fictional company called FitLoop), and you will be graded on both your individual
reasoning and your group's presentation.

**Total weight:** 20% of your final grade.

**The Sprint has three parts spread across the final weeks of the course:**

| Part | When | Format | Weight |
|---|---|---|---|
| Individual scenario quiz | Week 14 (in class) | Timed Canvas quiz, 60 min | 100 of 140 points |
| Group presentation | Week 15 (in class) | 12–15 min per group | 40 of 140 points |
| Group write-up | Week 16 (online) | Written submission, 2 pages max | — (see below) |

*Note: the write-up is part of the same 140-point assessment, submitted as a PDF to Canvas
by the end of Week 16. It is graded using the same group scorecard as the presentation.*

---

## Timeline

### Week 13 — Briefing released
One week before the Sprint work session, you receive the FitLoop company briefing (Part A
of the case scenario). **Read it carefully. Discuss it with your group.**

You will **not** receive model output data this week — that is released at the very start
of the Week 14 in-class session. There is nothing to run, no numbers to analyze, and no
model outputs to look at during Week 13. Your preparation this week is entirely conceptual:
understanding the two business decisions, thinking about which models are relevant, and
knowing which models you would *not* use and why.

### Week 14 — Individual scenario quiz + group work (in class)
- Model outputs are released at the start of class
- You have **60 minutes** to analyze the data and complete the individual scenario quiz
- The Canvas quiz closes automatically at the 60-minute mark — no extensions
- After submitting your quiz, the remaining class time is dedicated group work: refine
  your analysis, prepare your presentation outline, and align on your recommendation

### Week 15 — Group presentations (in class)
- Each group presents for 12–15 minutes
- No formal Q&A; the instructor and other students can ask questions freely
- Presentation order is randomized and announced on the day
- You can use any format: slides, whiteboard, verbal — whatever works for your group

### Week 16 — Group write-up due (online)
- A 2-page written summary of your group's analysis and recommendation
- Submitted as a PDF to Canvas by the deadline posted on the course site
- This is a clean write-up of what you presented, not new analysis

---

## The FitLoop Case and the Course Models

The Sprint case (FitLoop) uses **six pre-run model outputs** chosen because they connect
directly to the two business decisions in the scenario. The six outputs draw on models from
the first half of the course (Bayesian A/B testing, price elasticity, BG/NBD CLV, survival
analysis, uplift modelling, and Markov chains).

The individual assessment is a 10-question multiple-choice quiz in Canvas (Respondus
Lockdown Browser required). Each question presents a scenario with **specific numbers
from your group's model outputs** and asks you to identify the best analytical
interpretation or recommendation.

**You may have your group's model output packet open during the quiz.** You may not
communicate with group members while the quiz is open.

**The key feature of these questions:** All four answer choices are correct or plausible
statements about the FitLoop case. Your task is to identify the *best* answer given what
the specific question is asking. Some distractors are:
- Factually correct statements that are irrelevant to the question being asked
- Correct interpretations of a different model than the one the question is about
- True observations that lead to the wrong conclusion for the specific decision

This means you cannot answer the questions by looking up a definition or producing
a generic "best practice" statement. You need to understand what the numbers in *your
group's specific output* mean in context.

**Question topics** (one question per area):
1. Bayesian A/B test — interpreting posterior probability and credible intervals for a pricing decision
2. Price elasticity — using the elasticity coefficient alongside the A/B test for the same decision
3. BG/NBD CLV — P(alive) interpretation and retention campaign targeting
4. Survival analysis — Cox hazard ratios and the correct product intervention they imply
5. Uplift model — profit-maximising targeting threshold and sleeping dog exclusion
6. Markov chain — steady-state interpretation and identifying the highest-leverage bottleneck
7. Model selection — which model applies to which decision (and which does not)
8. Assumption identification — causal vs. observational inference in the Cox model
9. Integration — identifying the most important analytical gap in a proposed recommendation
10. Recommendation — selecting the recommendation that best integrates uncertainty with business constraints

---

## How to Prepare

**The course models you are expected to draw on:** You are expected to be familiar with
**all course models** — not just the six used in the pre-run outputs. The quiz includes a
question (Q7) about model selection that asks you to evaluate which model applies to which
decision and why a specific model *does not* apply in a given context. This means you need
to have surveyed the entire toolkit and understood the scope of each model.

---

## How to Prepare

**Before Week 13 (before the briefing is released):**

- Review the Model Reference Card — make sure you can sketch the estimand and one key
  assumption for each model covered in the course
- Know how to look up a specific model's output quickly in your group's analysis

**During Week 13 (after receiving the company briefing):**

- Read the briefing carefully — understand the two business decisions
- Discuss with your group which models are most relevant to each decision
- Think about which models you would *not* use and why (Q7 in the quiz covers model selection)
- **No model outputs are available yet** — your preparation this week is conceptual, not
  numerical. The numbers arrive at the start of Week 14.

**Week 14 (in-class):**

- The model outputs are pre-generated and provided to you — no coding required during the
  individual quiz time
- First 60 min: individual scenario quiz (Canvas MC, auto-closes)
- Remaining class time: group work — review the outputs together, align on your analysis,
  and prepare your presentation outline
- You can reference your group's model output packet during the quiz — it is open-book for
  the data, not for communication with group members while the quiz is open

**The best preparation:** Make sure you personally understand every number your group's
analysis produces. You cannot answer these questions by producing a generic statement —
each question requires you to integrate a specific number from your group's outputs with
the analytical reasoning the question is testing. If you let one group member do all the
analysis and you just watched, you will not have the depth needed to identify the best
answer from four plausible options.

---

## Group Presentation (Week 15)

**What we are looking for (40 points total):**

| Criterion | Points |
|---|---|
| Model selection — 2+ relevant models used, selection justified, at least one non-obvious exclusion reasoned | 15 |
| Output connection — recommendation explicitly tied to at least one specific number | 15 |
| Limitation identified — one genuine analytical gap named; not a data complaint but a specific unanswerable question | 10 |

**Practical details:**

- 12–15 minutes per group
- No strict structure — use whatever format communicates your analysis best
- The individual quiz is already submitted before any presentations begin,
  so what you say in the presentation does not affect your individual grade
- If your presentation disagrees with your quiz answers, that is fine — the quiz reflects
  your view at the time of analysis; the presentation is the group's collective view

---

## Group Write-Up (Week 16, due online)

A 2-page maximum written summary (PDF, submitted to Canvas):

- **Page 1:** The business context, your chosen models, and the specific findings
- **Page 2:** Your recommendation(s), the key uncertainty, and what would change your advice

The write-up is graded using the same three criteria as the presentation (model selection,
output connection, limitation identified). The higher of your presentation and write-up
score on each criterion is used — the write-up gives your group a second chance to
articulate any points that were unclear in the live presentation.

---

## Makeup Policy

**Individual scenario quiz (Week 14 Canvas quiz):**
If you cannot attend the in-class session in Week 14, you may request a makeup window
by emailing the instructor at least 24 hours in advance (or as soon as possible for
emergencies). The makeup quiz uses the same Canvas question set with a different time
window. It is taken independently with Respondus Lockdown Browser — no Codespace
access is required because no coding is needed during the quiz.

You can self-schedule the makeup by:
1. Emailing the instructor with your preferred 2-hour window (must be within 5 days
   of the original date)
2. The instructor will open the Canvas window for you
3. You take the quiz using LDB at the agreed time

**Group presentation (Week 15):**
If you cannot attend the presentation day, contact your group and the instructor.
Two options:
- **Option A:** Present at the end of the presentation session if schedule allows
- **Option B:** Record a 15-minute video of your group's presentation, submitted to
  Canvas by the end of Week 15 — this option requires all group members who were
  present for the original session to agree

If your entire group cannot present, schedule a makeup presentation during office hours
or a mutually agreed time before the Week 16 write-up deadline.

**Group write-up (Week 16):**
Standard late submission policy applies: −10% per day, up to 50% maximum penalty.
No submissions accepted more than one week after the deadline without prior arrangement.
