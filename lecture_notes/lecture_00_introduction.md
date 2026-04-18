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
- Know the 14 models covered and how they connect to real business decisions
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

Fourteen lectures. Each covers one model. The order follows mathematical dependency:
each model builds on concepts from previous ones.

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
| 12 | Difference-in-Differences | Can we establish causation from observational data? |
| 13 | Customer Segmentation | How do we group customers to target them differently? |
| 14 | CausalImpact | Did our marketing campaign actually cause the revenue lift? |

**The thread connecting all 14:** Every model addresses uncertainty about customer
behavior. Every model requires judgment about when it applies and when it doesn't.
None of them is a black box you can trust blindly.

---

### 1.4 Assessment

| Component | Weight | Description |
|---|---|---|
| Homework (14 × individual) | 50% | Part A autograded; Parts B+C autograded |
| Quiz 1 (after L06) | 5% | 20 T/F questions, Bayesian confidence scoring |
| Quiz 2 (after L12) | 5% | 20 T/F questions, Bayesian confidence scoring |
| Sprint | 20% | FitLoop case: six model outputs, group presentations |
| Checkpoints (in-class) | 20% | Participation credit; 5 questions per session |

**Collaboration policy:**
- Part A (math questions): Individual. No collaboration.
- Part B and C (code + interpretation): Collaboration permitted and encouraged.
- Quizzes: Individual. No resources.
- Sprint: Group presentation; individual scenario quiz submitted before group work begins.

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

**Step 3: Accept the GitHub Classroom invitation**

Your instructor will post an invitation link in Canvas. When you click it:
1. You will be asked to authorize GitHub Classroom
2. A private repository is automatically created for you
3. The repository is named: `hw01-bayesian-ab-[your-username]`

You will repeat this process for each of the 10 homework assignments.

---

### 2.3 Step-by-Step: Running Your First Codespace

**What is a Codespace?**

A Codespace is a virtual computer that runs in Microsoft's cloud, accessed entirely
through your browser. When you open a Codespace:
- A Linux machine with 4 CPUs and 8 GB RAM starts up
- Python 3.10 is pre-installed
- All required libraries (numpy, pandas, scipy, lifetimes, prophet, etc.) are
  pre-installed via the repository's `devcontainer.json`
- VS Code runs in the browser as your editor

You pay nothing. GitHub provides 120 core-hours per month free, which is approximately
60 hours of 2-core Codespace time — more than enough for this course.

**Opening a Codespace for the first time:**

1. Go to your homework repository on GitHub
2. Click the green **Code** button
3. Click the **Codespaces** tab
4. Click **Create codespace on main**
5. Wait 2–3 minutes for the environment to build (first time only; subsequent opens
   take about 30 seconds)
6. VS Code opens in your browser with the homework notebook visible in the file explorer

**Opening the Jupyter notebook:**

Once VS Code is open:
1. In the left file panel, click `homework_01_bayesian_ab.ipynb`
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

Submission is automatic when you push to GitHub. The autograder runs within 2 minutes.

**Step 1:** Complete all cells in the notebook. Every `q_N_variable = None` line
should have a real value (not None) before you submit.

**Step 2:** Run all cells in order to verify there are no errors.
In VS Code: click **Run All** (the ▶▶ button at the top of the notebook).

**Step 3:** In VS Code's source control panel (the branch icon in the left sidebar):
1. Click the **+** next to your changed files to stage them
2. Type a commit message: `HW01 final submission`
3. Click **Commit**
4. Click **Sync Changes** (or the push icon)

**Step 4:** Go to your repository on GitHub. Click the **Actions** tab. You will
see your autograder run. Green checkmark = passed. Red X = at least one Part A
question is wrong or the notebook had an error.

**Reading autograder output:**
Click on the failed run → click on the "grade" job → expand "Run Part A autograder"
to see which specific questions failed and by how much.

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
| **GitHub README** | ✅ Partial (display math only) | Documentation |

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

## PART 1 Checkpoint

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

5. You estimate a Cox proportional hazards model and the concordance index (C-index)
   is 0.82. A teammate says "great, the model explains 82% of the variation." What is
   wrong with this interpretation, and what does 0.82 actually mean?

---

### Checkpoint Answer Key

**Q1.** Analyst track: you are running the model. You care about whether the
assumptions hold, how to diagnose violations, and how to report uncertainty correctly.
Example: you fit a Cox model and notice the Schoenfeld residuals are correlated with
time — you flag a PH assumption violation and report the issue.
Manager track: you are acting on the model output. You care about whether the
recommendation is trustworthy and what would change it. Example: you receive a Cox
model output showing HR=0.74 for workouts and ask: "Is this finding stable if we
remove new customers from the sample? Could early dropout be driving this?"

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

**Q5.** Wrong: C-index ≠ R². R² measures percentage of variance explained in a
regression. C-index (concordance index) measures the fraction of comparable pairs the
model ranks correctly by predicted survival time. C=0.50 means no better than random
(coin flip). C=0.82 means the model correctly ranks 82% of pairs where one customer
churned before the other. It is a ranking metric, not a variance explanation metric.

