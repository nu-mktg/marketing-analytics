# Lecture 0: Introduction to Marketing Analytics
## Course Overview, Philosophy, and Technology Setup

---

### Overview

This is the first class session. No mathematics today. Instead: why this course exists,
what kind of thinking it trains, and how to get your development environment working
before next class.

**By the end of this session you should:**
- Understand the course philosophy and what distinguishes analytical judgment from
  analytical execution
- Know the ten models covered and how they connect to real business decisions
- Have a working GitHub Codespace running a Jupyter notebook
- Have submitted your first homework (a technology setup verification)

---

## PART 1: Course Philosophy

---

### 1.1 The Strategic Frame: Why Analysts Fail — and Why They Shouldn't

Most analytics programs teach you how to run models. This course teaches you when to
trust them.

The distinction matters. An analyst who can fit a Cox proportional hazards model is
useful. An analyst who can fit the model AND explain why the proportional hazards
assumption might be violated for subscription age AND know what to do when it is —
that person is rare and valuable.

We draw on practitioners from Lenny Rachitsky's podcast — a consistent source of
honest, detailed accounts of how analytics actually works at high-growth companies.
The practitioners we reference have collectively managed:
- Over $3.5 billion in paid acquisition budgets (Jonathan Becker)
- 600+ A/B experiments on a single product feature (Jackson Shuttleworth, Duolingo)
- The data organization at DoorDash from startup to public company (Jess Lachs)
- Growth teams at Netflix, Grammarly, Evernote, Shopify, and Airtable

The pattern across all of them: the model output is rarely the hard part. What is
hard is knowing which model to run, what assumptions it makes, when those assumptions
are violated, and how to communicate the results to people who will act on them.

---

### 1.2 The Two Tracks

This course is designed for two types of students simultaneously:

**Analyst track:** You will implement models, interpret outputs, and check whether
assumptions hold. You need to know the math well enough to recognize when a model
is being misapplied — even when the code runs without errors.

**Manager track:** You will commission analyses, receive model outputs, and make
decisions based on them. You need to know enough to ask the right questions:
"What assumption does this model make that might not hold here?" and "What would
change your recommendation?"

Both tracks are present in every session. The checkpoint questions are designed to
serve both. The homework has individual math questions (analyst track) and
interpretation questions where collaboration is encouraged (manager track).

---

### 1.3 The Course Structure

Twelve lectures: this introductory session (Lecture 0) and eleven model lectures,
L01 through L11. Each of the eleven covers one model, and the order follows
mathematical dependency — each model builds on concepts from previous ones.

| Lecture | Model | Core question answered |
|---|---|---|
| 1 | Bayesian A/B Testing | Did this change actually work? |
| 2 | Price Elasticity / OLS | How sensitive are customers to price? |
| 3 | Marketing Mix Modeling | Which marketing channels actually drive revenue? |
| 4 | Survival Analysis | When will customers leave — and who is most at risk? |
| 5 | Markov Chains | Where are customers going, and how do we change the trajectory? |
| 6 | Multi-Touch Attribution | Which channels deserve credit for conversions? |
| 7 | Uplift Modeling | Which customers will actually change behavior because of our campaign? |
| 8 | Customer Lifetime Value | How much is a customer worth — and are they still there? |
| 9 | Conjoint Analysis | What do customers value — and how much will they pay? |
| 10 | Demand Forecasting | Where is our business heading? |
| 11 | Synthesis & Capstone | How do the models fit together into a decision framework? |

**The thread connecting all of them:** Every model addresses uncertainty about
customer behavior. Every model requires judgment about when it applies and when it
doesn't. None of them is a black box you can trust blindly.

> **Not covered this term.** The course materials also contain notes and notebooks for
> Difference-in-Differences, Customer Segmentation and CausalImpact. They are not
> taught this term, carry no homework and are not assessed. They are there as
> reference if you want to read ahead.

---

### 1.4 Assessment

| Component | Weight | Description |
|---|---|---|
| Homework (×11) | 40% | 11 graded assignments, equal weight |
| Quiz 1 (Week 7, covers L01–L05) | 10% | In-class, 20 T/F questions, Bayesian confidence scoring |
| Quiz 2 (covers L06–L10) | 10% | In-class, 24 T/F questions, Bayesian confidence scoring |
| Class Participation | 10% | Attendance and contribution to class discussion |
| Sprint — Individual | 15% | Individual 10-question Canvas MC quiz (auto-graded) |
| Sprint — Group | 15% | FitLoop case: group presentation and write-up |

> Canonical as of 2026-09-04 (sh.lee). The rows above are byte-identical in
> `instructor/course_syllabus_template.md` and `lecture_notes/lecture_00_introduction.md`
> by requirement; if they ever disagree, one of them is wrong.
> Two reconciliations are recorded here. **2026-08-12:** this table previously read
> HW 50 / Q 5+5 / Sprint 20 / Checkpoints 20, which contradicted the syllabus; the
> syllabus version won, because it puts ~35% of the grade on individually-discriminating
> work rather than ~10–15%, which matters when homework is AI-assisted. **2026-09-04:**
> the course was scoped to L00–L11, so Homework went ×14 → **×11** (hw01–hw11), and the
> **Checkpoints** row became **Class Participation** — checkpoint questions are now
> worked through together in class and are not submitted or graded. **Also 2026-09-04:**
> Quiz 2 **lost its week number**, because the real Fall-2026 calendars put it in week 12 for
> MKTG3501 and week 11 for MKTG6434. Quiz 1 is week 7 in both and keeps its week. Do not
> re-add a week to Quiz 2 — any single number is wrong for one of the two courses. Dates:
> `instructor/course_schedule_template.md`.

**Collaboration policy:**
- Part A (math questions): Individual. No collaboration.
- Part B and C (code + interpretation): Collaboration permitted and encouraged.
- Quizzes: Individual. No resources.
- Sprint: Group presentation; individual scenario quiz submitted before group work begins.

**Checkpoints are not graded.** The five questions at the end of each lecture are not
submitted. We work through them together in class, and you should attempt them
beforehand — you will be asked to **explain your reasoning, not just state the
answer.** Using Copilot or Claude to reach the answer is expected; what you cannot
outsource is the explanation. The 10% in the table above is **Class Participation**,
awarded for attendance and contribution to class discussion; the criteria are in the
syllabus.

---

## PART 2: Technology Setup

---

### 2.1 The Stack

You need exactly three tools. All are free. None require installation on your personal
computer.

| Tool | What it is | What you use it for |
|---|---|---|
| **GitHub** | Code hosting and version control | Receiving assignments; submitting homework |
| **GitHub Codespaces** | Cloud-hosted development environment | Running Python code without any local setup |
| **Jupyter Notebook** | Interactive Python environment | Writing and running homework code |

Everything runs in the browser. You do not need to install Python, Jupyter, or any
data science library. The Codespace has everything pre-configured.

---

### 2.2 Step-by-Step: Setting Up GitHub

**Step 1: Create a GitHub account**

1. Go to github.com
2. Click "Sign up"
3. Use your university email address
4. Choose the free plan
5. Verify your email

**Step 2: Accept the GitHub Education benefits (optional but recommended)**

1. Go to education.github.com/students
2. Click "Get student benefits"
3. Upload a photo of your student ID
4. This gives you GitHub Pro for free (larger Codespaces quota)

**Step 3: Create your course repository from the template**

You own **one private repository** for the whole course, and every homework arrives
in it as its own folder (`hw00/`, `hw01/`, `hw02/`, and so on). You do not create a
new repository per assignment.

1. Go to `github.com/shleeneu/marketing-analytics`
2. Click the green **Use this template** → **Create a new repository**
3. Owner: **your own account**
4. Repository name: `marketing-analytics` — type it exactly
5. Visibility: **Private**
6. Click **Create repository**

> ⚠️ The name must end with `marketing-analytics`. Homework is delivered to
> repositories with that name. A different name means the assignment never
> arrives, and nothing will look broken when it doesn't.

**Step 4: Install the course app on your repository**

1. Go to `github.com/apps/nu-mktg-grader/installations/new`
2. Choose **Only select repositories**
3. Select the repository you just created → **Install**

The repository is yours, so the course cannot reach it by default. The app is how
homework gets delivered to you and how your feedback gets written back. It can read
and write repository contents and nothing else — it cannot see your other
repositories, and you can remove it at any time.

Confirm at `github.com/settings/installations`: it should list **NU-mktg-grader**
with your repository under it.

Steps 3 and 4 are done once, at the start of term. After that, each new homework
folder arrives in the repository you already have.

---

### 2.3 Step-by-Step: Running Your First Codespace

**What is a Codespace?**

A Codespace is a virtual computer that runs in Microsoft's cloud, accessed entirely
through your browser. When you open a Codespace:
- A Linux machine with 2 CPUs and 8 GB RAM starts up
- Python 3.11 is pre-installed
- All required libraries (numpy, pandas, scipy, lifetimes, prophet, etc.) are
  pre-installed via the repository's `devcontainer.json`
- VS Code runs in the browser as your editor

You pay nothing. GitHub provides 120 core-hours per month free, which is approximately
60 hours of 2-core Codespace time — more than enough for this course.

You create **one** Codespace and reopen it for the whole course — your repository
holds every homework, so one Codespace serves all of them.

**Opening a Codespace for the first time:**

1. Go to your course repository on GitHub
2. Click the green **Code** button
3. Click the **Codespaces** tab
4. Click **Create codespace on main**
5. Wait 2–3 minutes for the environment to build (first time only; subsequent opens
   take about 30 seconds)
6. VS Code opens in your browser with the homework notebook visible in the file explorer

**Opening the Jupyter notebook:**

Once VS Code is open:
1. In the left file panel, open the `hw01/` folder and click
   `homework_01_bayesian_ab.ipynb`
2. The notebook opens in the editor pane
3. If prompted "Select Kernel": choose **Python 3 (ipykernel)**
4. You are ready to work

**Stopping a Codespace (IMPORTANT):**

Codespaces continue consuming your free hours if left running. Always stop them when
done:
1. Click the **...** menu in the lower-left of VS Code
2. Click **Stop Current Codespace**
Alternatively: go to github.com/codespaces and stop it from the dashboard.

---

### 2.4 Jupyter Notebook: Quick Reference

A Jupyter notebook is a document containing cells. Each cell is either:
- **Code cell:** Python code that runs when you press Shift+Enter
- **Markdown cell:** Formatted text (for instructions and explanations)

**Essential keyboard shortcuts:**

| Action | Shortcut |
|---|---|
| Run current cell | Shift + Enter |
| Run current cell, stay | Ctrl + Enter |
| Insert cell below | B (in command mode) |
| Delete cell | DD (in command mode) |
| Toggle command mode | Escape |
| Toggle edit mode | Enter |
| Run all cells | Kernel → Restart & Run All |
| Find variable value | Type the variable name in a cell and run it |

**Command mode vs Edit mode:**
- **Edit mode** (green border): you are typing inside a cell
- **Command mode** (blue border): keyboard shortcuts work; you are navigating between cells
- Press Escape to enter command mode; press Enter to enter edit mode

**The most important rule:** Always run cells in order from top to bottom. If a later
cell depends on a variable set in an earlier cell, running them out of order will cause
errors. When in doubt: Kernel → Restart & Run All.

---

### 2.5 Submitting Homework

> **Answers:** homework answers are not duplicated here. The single source of truth is
> `answer_keys/hwNN.json` in the instructor repo, synced to the private key repo with
> `./tools/push_answer_keys.sh`. Duplicating them in this file is what caused them to
> drift out of sync with the notebooks (Task 015).


**Getting the assignment.** New homework is delivered into the repository you
already own. In your Codespace terminal, from the repository root:

```bash
git pull
```

A new `hwNN/` folder appears. That is the whole delivery step.

**Step 1:** Complete all cells in the notebook. Every `q_N_variable = None` line
should have a real value (not None) before you submit.

**Step 2:** Run all cells in order to verify there are no errors, from a fresh
kernel: **Kernel → Restart & Run All Cells**. It must run clean.

**Step 3:** Submit by pushing. In the Codespace terminal, from the repository root:

```bash
git add .
git commit -m "completed hw01"
git push
```

`add` → `commit` → `push` is one sequence and you run all three every time.
**Nothing is submitted until you push.** (The same thing can be done from VS Code's
source control panel: the branch icon in the left sidebar → **+** to stage → type a
message → **Commit** → **Sync Changes**.)

**Step 4:** Read your feedback. Wait a couple of hours, then:

```bash
git pull
```

Your score and a per-question breakdown are written into your repository as
`hwNN/GRADE.md`.

**How your work gets graded.** After you push, the course's grading system picks up
your latest commit, grades it, and writes `hwNN/GRADE.md` back into your repository.
Grading runs on the course's own infrastructure, not in your repository. It sweeps
every couple of hours, so allow some time — and push early rather than once at the
deadline, since every push is regraded. `GRADE.md` names the commit it graded, so you
can always tell whether the feedback you are reading is for your latest work.

**`hw00` is ungraded and produces no `GRADE.md`.** If none appears for hw00, nothing
is wrong.

**If `git push` is rejected,** new homework was added to your repository since you
last pulled. Normal, and not something you did wrong:

```bash
git pull
git push
```

---

### 2.6 LaTeX for Digital Mathematics

This section is for students who want to write mathematical derivations digitally
instead of on paper or a whiteboard. This is entirely optional for the course but
highly recommended if you plan to continue working with quantitative methods.

**Why LaTeX?**

LaTeX is the universal language for mathematical notation. Every academic paper in
statistics, economics, and computer science uses it. Once you learn it, you can write
any formula in any document, on any device, in a way that renders beautifully and is
100% searchable, versionable, and readable by any collaborator.

**The syntax you need for this course (90% of all formulas):**

```latex
# Inline math (renders inline with text):
$\hat{\beta}_1 = \frac{\sum(x_i - \bar{x})(y_i - \bar{y})}{\sum(x_i - \bar{x})^2}$

# Display math (rendered on its own line, centered):
$$\hat{S}(t) = \prod_{j: t_j \leq t} \frac{n_j - d_j}{n_j}$$

# Greek letters:
\alpha \beta \lambda \epsilon \theta \pi \sigma \mu \tau

# Operations:
\sum_{i=1}^{n}    % summation
\prod_{j=1}^{k}   % product
\frac{a}{b}       % fraction
\sqrt{x}          % square root
\int_0^1          % integral

# Decorators:
\hat{\beta}        % hat (estimate)
\bar{x}            % bar (mean)
\tilde{x}          % tilde
\mathbf{X}         % bold (matrices/vectors)

# Subscripts and superscripts:
x_{ij}             % subscript
e^{\beta^\top x}   % superscript

# Parentheses (auto-sizing):
\left( \frac{a}{b} \right)
```

**Where to use it:**

| Platform | LaTeX support | Recommended use |
|---|---|---|
| **Jupyter notebook** | ✅ Native — type `$formula$` in a Markdown cell | Homework work, derivations |
| **Overleaf** (browser) | ✅ Native, collaborative | Polished reports |
| **GitHub README** | ✅ Native — inline `$...$` and block `$$...$$` | Documentation |

**Practice exercise (do now, in your Codespace):**

Open a new Jupyter notebook in your Codespace. Create a Markdown cell and type:

```markdown
## The OLS Slope Formula

The ordinary least squares slope estimator is:

$$\hat{\beta}_1 = \frac{\sum_{i=1}^n (x_i - \bar{x})(y_i - \bar{y})}{\sum_{i=1}^n (x_i - \bar{x})^2}$$

This is the ratio of the sample covariance of $x$ and $y$ to the sample variance of $x$.
```

Press Escape, then Shift+Enter to render it. You have just written professional
mathematical notation.

**30-minute learning path:**

1. (5 min) Read: overleaf.com/learn/latex/Mathematical_expressions — the free tutorial
2. (10 min) Practice: type all 10 formulas from this course into a Markdown cell in Jupyter
3. (15 min) Write the Beta posterior update derivation from Lecture 1 in LaTeX

After this practice session, you will be fluent enough for everything in this course.

---

### 2.7 Required Tools Summary

**Required (free):**
- GitHub account at github.com
- GitHub Codespaces (accessed through GitHub — no installation)

**Optional (for those who want to go deeper):**
- **Quarto** (quarto.org) — builds on R Markdown to create slides, PDFs, and HTML documents
  - Already used to generate this course's materials
  - Great for producing polished reports from your analyses
- **Overleaf** (overleaf.com) — browser-based LaTeX editor, free tier, real-time collaboration

---

## PART 3: Course Norms

---

### 3.1 On Using AI

You may use AI assistants (Claude, ChatGPT, Copilot) for Part B and Part C of homeworks.
You may not use them for Part A.

The rule is not arbitrary. Part A tests whether you can perform the mathematical
operations by hand. No professional analyst needs to memorize the OLS formula — but
every professional analyst needs to know what the formula is doing well enough to
check whether an AI-generated answer is plausible. Part A is the training for that
judgment.

A more general principle for this course: use AI to accelerate computation and
drafting. Use your own judgment to evaluate results. The goal is that by the end of
the course, you can look at any model output and say — with confidence — whether
it is correct, approximately correct, or wrong and why.

### 3.2 On Large Language Models and Marketing Analytics

One of the most important skills this course develops is knowing when to trust a
model's output. This applies equally to statistical models and to language models.

Claude (and similar LLMs) will confidently state wrong posterior distributions,
incorrect elasticity calculations, and misidentified hazard ratios. They make
specific numerical errors with no warning or uncertainty. If you ask Claude to compute
a Kaplan-Meier curve by hand, you should verify the answer against your own calculation.

This is not a limitation unique to AI. It applies to any tool: Excel formulas, R
packages, Python libraries. The discipline of verification — of developing enough
quantitative literacy to check the tool's output — is what this course is building.

### 3.3 On Struggle

The material in this course is genuinely difficult in places. The OLS derivation,
the partial likelihood for Cox regression, and the Shapley value enumeration are all
multi-step algebraic arguments. The first time you see them, they will probably not be
obvious.

This is normal and expected. The checkpoint questions are designed to reveal when
something is unclear so we can address it in class, not to grade you on first-pass
comprehension. Come prepared to be uncertain, ask questions, and revise your
understanding.

---

## Checkpoint

These questions have no math. They test whether you understand the course structure
and have completed the technology setup.

1. In your own words: what is the difference between the "analyst track" and the
   "manager track" framing of this course? Give a concrete example of how the same
   model output would be used differently by each.

2. Which collaboration rule applies to Homework Part A — individual or permitted?
   Which rule applies to Part C?

3. Describe the three steps to stop a Codespace when you are done working, and why
   stopping is important.

4. In Jupyter, what is the keyboard shortcut to run the current cell and move to the
   next? What does it mean to be in "command mode" vs "edit mode"?

5. This course expects you to use an AI assistant for Parts B and C, but not Part A.
   In your own words: why is Part A the exception? And give one concrete example of an
   AI-generated answer you would still need to check before submitting it.

---

### Checkpoint Answer Key

**Q1.** Analyst track: you are running the model. You care about whether the
assumptions hold, how to diagnose violations, and how to report uncertainty correctly.
Example: you fit a model to compare two campaigns and notice one group is much smaller
than the other — you check whether the difference is distinguishable from noise before
reporting it as a result.
Manager track: you are acting on the model output. You care about whether the
recommendation is trustworthy and what would change it. Example: you are handed a result
saying customers who use a feature churn less, and you ask: "Does using the feature keep
them, or do the customers who were staying anyway use it more?"

*(Examples here are deliberately model-free: this is Lecture 0 and no model has been
taught yet. The specific techniques arrive from Lecture 1 onward.)*

*Common wrong answer:* "The analyst does the math and the manager reads the answer."
This misses the point — both tracks require understanding what the math means, just
from different angles.

**Q2.** Part A: Individual — no collaboration. Part C: Collaboration permitted.
(Parts B and C are collaboration-permitted, and both are autograded.)

*Common wrong answer:* Collaboration is never permitted. Incorrect — the distinction
is specifically about Part A (individual math) vs. Parts B+C (collaborative applied
work).

**Q3.** (1) Click **...** in the lower-left VS Code status bar. (2) Select "Stop
Current Codespace." (3) Alternatively: go to github.com/codespaces and stop it from
the dashboard. Importance: Codespaces consume your free tier hours (120/month) even
when you are not actively typing. Leaving one running overnight wastes approximately
8–16 hours of quota.

**Q4.** Shift+Enter runs the current cell and advances to the next. Command mode
(blue border): keyboard shortcuts are active; you navigate between cells. Edit mode
(green border): you are typing inside a cell. Press Escape for command mode; press
Enter or click inside a cell for edit mode.

> **Also asked on the slides:** *"You get `NameError: name 'q5' is not defined`. What
> went wrong?"* — You ran the cells out of order. Always run cells in order from top to
> bottom: if a later cell depends on a variable set in an earlier cell, running them out
> of order will cause errors. Here the cell that defines `q5` was never run, or was run
> only after the cell that uses it. Fix: Kernel → Restart & Run All.

**Q5.** Part A tests whether you can perform the mathematical operations yourself. The
point is not memorisation — no professional analyst memorises the OLS formula — but that
you understand what the formula is doing well enough to judge whether an AI-generated
answer is plausible. Part A is the training for that judgement, so outsourcing it
removes the only thing it measures. Parts B and C are where AI genuinely accelerates
the work, because there you are evaluating output rather than producing it.

Any concrete example of something checkable earns full credit. Good ones: a number that
is the right magnitude but the wrong sign; an answer that ignores a stated constraint in
the question; confidently named "results" from a dataset column that does not exist; a
formula applied to the wrong variable. The habit being built is the course's whole
premise — the agent can interpret, propose and flag; **you** verify, catch its errors,
and own the result.

*Common wrong answer:* "Because using AI on Part A is cheating." True but circular — the
question asks WHY the rule exists, and the reason is that Part A is the only part that
builds the judgement you need to supervise an AI everywhere else.

> **Also asked on the slides:** *"P(B>A) = 0.87. Your manager says 'B wins 87% of
> repeated experiments.' Is this correct?"* — No. P(B > A) = 0.87 is a **Bayesian
> posterior probability** about the unknown true rates, given this data and this prior.
> It does not say how often B would win in repeated experiments — that would be a
> frequentist statement about long-run sampling, which is not what Bayesian inference
> computes. The correct reading: given what we observed, there is an 87% probability that
> B's true conversion rate is higher than A's. Lecture 1 works this through
> in full with P(B > A) = 0.91.

