# Sprint Assessment Guide
## Marketing Analytics — Student Reference

---

## What Is the Sprint?

The Sprint is the capstone assessment of the course. It is a structured case analysis that
brings together the course's analytical models. You will work as a group on two business
decisions at FitLoop, a fictional fitness-subscription company, and you will be graded on both your individual
reasoning and your group's presentation.

**Total weight:** 30% of your final grade (Sprint — Individual 15% + Sprint — Group 15%).

**The Sprint has three parts spread across the final weeks of the course:**

| Part | When | Format | Canvas points | Weight |
|---|---|---|---|---|
| Individual scenario quiz | Mon 11/30 (in class) | Timed Canvas quiz, 60 min | 100 | 15% |
| Group presentation | In class, from Mon 12/7 | 10–15 min per group | 40 | 15% |
| Group write-up | Due Mon 12/14 (online) | Written submission, 2 pages max | scored with the presentation | (included in the 15%) |

**How the points map to your grade.** The two point totals are *separate Canvas scales*, not
shares of one 140-point pot: your individual quiz score out of 100 becomes the Individual 15%,
and your group's scorecard score out of 40 becomes the Group 15%. So a student who scores 80/100
on the quiz and whose group scores 30/40 earns (0.80 × 15) + (0.75 × 15) = **23.25% of the
course grade** out of the 30% the Sprint is worth. The larger quiz point total does **not** make
the individual portion count for more than the group portion — the split is 50/50.

*Note: the write-up is submitted as a PDF to Canvas by Mon 12/14 and is graded using the
same 40-point group scorecard as the presentation; the higher of your presentation and write-up
score on each criterion is the one that counts.*

---

## Timeline

Your section's exact dates, including any extra work or presentation days, are on its
**Sprint Schedule** page in Canvas.

### Mon 11/23 — Briefing released
One week before the Sprint work session, you receive the FitLoop company briefing (Part A
of the case scenario). **Read it carefully. Discuss it with your group.**

You will **not** receive model output data this week — that is released at the very start
of the 11/30 in-class session. There is nothing to run, no numbers to analyze, and no
model outputs to look at before 11/30. Your preparation this week is entirely conceptual:
understanding the two business decisions, thinking about which models are relevant, and
knowing which models you would *not* use and why.

### Mon 11/30 — Individual scenario quiz + group work (in class)
- Model outputs are released at the start of class
- You have **60 minutes** to analyze the data and complete the individual scenario quiz
- The Canvas quiz closes automatically at the 60-minute mark — no extensions
- After submitting your quiz, the remaining class time is dedicated group work: refine
  your analysis, prepare your presentation outline, and align on your recommendation

### From Mon 12/7 — Group presentations (in class)
- Your section's presentation dates are on its Sprint Schedule page in Canvas
- Each group presents for 10–15 minutes
- No formal Q&A; the instructor and other students can ask questions freely
- Presentation order is randomized and announced on the day
- You can use any format: slides, whiteboard, verbal — whatever works for your group

### Mon 12/14 — Group write-up due (online)
- A 2-page written summary of your group's analysis and recommendation
- Submitted as a PDF to Canvas by the deadline posted on the course site
- This is a clean write-up of what you presented, not new analysis

---

## The FitLoop Case and the Course Models

The Sprint case (FitLoop) uses **six pre-run model outputs** chosen because they connect
directly to the two business decisions in the scenario. The six outputs use Bayesian A/B testing (L01),
price elasticity (L02), survival analysis (L04), Markov chains (L05), uplift modeling (L07) and
BG/NBD CLV (L08). MMM (L03) is tested in Quiz 1, and Shapley attribution (L06), conjoint analysis (L09) and
Prophet (L10) are tested in Quiz 2. These four models enter the Sprint through the group presentation and write-up,
whose last rubric criterion asks for a follow-up analysis that uses one of them.

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
5. Uplift model — profit-maximizing targeting threshold and sleeping dog exclusion
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

**Before 11/23 (before the briefing is released):**

- Review the Model Reference Card — make sure you can sketch the estimand and one key
  assumption for each model covered in the course
- Know how to look up a specific model's output quickly in your group's analysis
- Work the Kibblewick practice case with your Sprint group. It is ungraded, uses the Sprint's
  format, and its three pages are on Canvas. Your section's Sprint Schedule page says when you
  work it. We go over it in class on 11/23

**Between 11/23 and 11/30 (after receiving the company briefing):**

- Read the briefing carefully — understand the two business decisions
- Discuss with your group which models are most relevant to each decision
- Think about which models you would *not* use and why (Q7 in the quiz covers model selection)
- **No model outputs are available yet** — your preparation this week is conceptual, not
  numerical. The numbers arrive at the start of class on 11/30.

**Mon 11/30 (in class):**

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

## Group Presentation (in class, from Mon 12/7)

**What we are looking for (40 points total):**

| Criterion | Points |
|---|---|
| Model selection — 2+ relevant models used, selection justified, at least one non-obvious exclusion reasoned | 15 |
| Output connection — recommendation explicitly tied to at least one specific number | 15 |
| Limitation and follow-up: one genuine analytical gap named, plus the follow-up analysis that would answer it, using a model from L03, L06, L09 or L10 (for example, conjoint willingness to pay for the price decision): what you would run, on which data, and how the result could change your recommendation | 10 |

**Practical details:**

- 10–15 minutes per group; nothing after 15:00 is scored
- No strict structure — use whatever format communicates your analysis best
- Where the briefing gives no information, you may state an assumption and use it, labeled as an
  assumption. Every number behind your recommendation comes from the briefing or your group's packet.
- For the follow-up, state which result would change your recommendation and how. A result for an
  analysis your group has not run receives no credit.
- The individual quiz is already submitted before any presentations begin,
  so what you say in the presentation does not affect your individual grade
- If your presentation disagrees with your quiz answers, that is fine — the quiz reflects
  your view at the time of analysis; the presentation is the group's collective view

---

## Group Write-Up (due Mon 12/14, online)

A 2-page maximum written summary (PDF, submitted to Canvas):

- **Page 1:** The business context, your chosen models, and the specific findings
- **Page 2:** Your recommendation(s), the key uncertainty, and what would change your advice

The write-up is graded using the same three criteria as the presentation (model selection,
output connection, limitation and follow-up). The higher of your presentation and write-up
score on each criterion is used — the write-up gives your group a second chance to
articulate any points that were unclear in the live presentation.

---

## Absences and Late Work

**Quiz session and presentations:** If you cannot attend the quiz session or a presentation
session, contact the instructor before the session.

**Group write-up (due Mon 12/14):**
The course's single late policy applies, exactly as it does to homework:
**on time = full credit · within 24 hours of the deadline = half credit** (half the score
you earn) **· more than 24 hours late = not accepted.**
