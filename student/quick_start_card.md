# Student Quick-Start Card
## Marketing Analytics — Getting to Your First Submission

---

## One repository for the whole course

You own **one private repository** for this course. Every homework arrives in it as
its own folder:

```
your-repo/
├── hw00/     ← setup verification (ungraded)
├── hw01/     ← arrives when hw01 is released
├── hw02/
└── ...
```

You do not create a new repository per assignment, and you do not need a new
Codespace per assignment.

---

## Setup — once, at the start of term

```
Step 1: Create your repository
         → Go to github.com/shleeneu/marketing-analytics
         → Green "Use this template" → "Create a new repository"
         → Owner: YOUR account
         → Repository name: marketing-analytics   ← type it exactly
         → Visibility: Private
         → "Create repository"

         ⚠️ The name must end with "marketing-analytics". Homework is
            delivered to repositories with that name. A different name
            means the assignment never arrives, and nothing will look
            broken when it doesn't.

Step 2: Install the course app on your repository
         → Go to github.com/apps/nu-mktg-grader/installations/new
         → Choose "Only select repositories"
         → Select the repository you just created → Install

         Why: the repository is yours, so the course cannot reach it by
         default. The app is how homework gets delivered to you and how
         your feedback gets written back. It can read and write repository
         contents and nothing else — it cannot see your other
         repositories, and you can remove it at any time.

         Confirm: github.com/settings/installations should list
         NU-mktg-grader with your repository under it.

Step 3: Open your Codespace
         → Your repo on GitHub → green "Code" button → Codespaces tab
         → "Create codespace on main"
         → First build takes a few minutes (it installs Python 3.11 and
           every course library at pinned versions)

         You only ever create ONE. Your repository holds every homework,
         so one Codespace serves the whole course. After this you
         REOPEN it rather than creating another.

         ⚠️ The editor opens BEFORE the install finishes. Libraries keep
            downloading in the background for the first few minutes.
            If hw00 says a library is "not found", wait 2-3 minutes and
            run it again — don't rebuild, don't pip install.

Step 4: Run hw00
         → Open hw00/homework_00_introduction.ipynb
         → Kernel → Restart & Run All Cells
         → No errors = you are set up
```

---

## Every homework after that

```
Step 1: Get the assignment
         In your Codespace terminal:   git pull
         A new hwNN/ folder appears.

Step 2: Do the work
         → Open hwNN/homework_NN_*.ipynb
         → Replace each None with your answer
         → Kernel → Restart & Run All Cells
           (it must run clean from a fresh kernel)

Step 3: Submit
         git add .
         git commit -m "completed hw01"
         git push

Step 4: Read your feedback
         Wait a couple of hours, then:   git pull
         Your score and per-question breakdown are in hwNN/GRADE.md
```

### How your work gets graded

After you push, the course's grading system picks up your latest commit, grades
it, and writes your feedback into your repository as `hwNN/GRADE.md`. It runs
every couple of hours, so allow some time — and push early rather than once at
the deadline, since every push is regraded.

`GRADE.md` names the commit it graded, so you can always tell whether the feedback
you are reading is for your latest work.

**`hw00` is ungraded and produces no `GRADE.md`.** If none appears for hw00,
nothing is wrong.

---

## Git, in the four commands you actually need

**Your whole repository is ONE git repository.** The `hwNN/` folders are just
folders inside it, not separate projects — so you run git from the repository
root, which is where the Codespace terminal already opens.

| command | what it does |
|---|---|
| `git pull` | brings *down* anything new — this is how a new homework folder appears |
| `git add .` | stages your changes: marks them to be included in the next save |
| `git commit -m "completed hw01"` | saves a snapshot **locally**, inside your Codespace |
| `git push` | sends your commits *up* to GitHub — **nothing is submitted until you push** |

`add` → `commit` → `push` is one sequence, and you run all three every time.
`git status` shows what you changed; `git log --oneline` shows what you already
committed.

**If `git push` is rejected,** new homework was added to your repository since you
last pulled. Normal, and not something you did wrong:

```bash
git pull
git push
```

---

## Keyboard Shortcuts You Will Use Every Day

| Action | Shortcut |
|---|---|
| Run current cell | `Shift + Enter` |
| Run cell, stay on it | `Ctrl + Enter` |
| Restart kernel and run all | Menu: Kernel → Restart & Run All |
| Save notebook | `Ctrl + S` |
| Open terminal | `Ctrl + `` ` (backtick) |

---

## When Things Go Wrong

| Problem | Fix |
|---|---|
| `NameError: q5 is not defined` | Kernel → Restart & Run All Cells |
| Cell runs forever | Kernel → Interrupt |
| The new homework isn't in my repo | `git pull`. Still missing? Check that NU-mktg-grader is installed at github.com/settings/installations **and** that your repo name ends with `marketing-analytics`. |
| No `GRADE.md` appeared after a day | Confirm you actually **pushed** (`git log origin/main --oneline`), and that the app is still installed. hw00 never produces one. |
| `GRADE.md` looks like old feedback | Check the commit it names at the top — if it isn't your latest, the next sweep hasn't run yet. |
| "Module not found" in a brand-new Codespace | It is still installing. Wait 2–3 minutes and re-run the cell — the first cell of hw00 tells you when this is what's happening. |
| "Module not found" after waiting | Rebuild the Codespace (Codespaces → `...` → Rebuild container) rather than `pip install`ing, so your versions still match what the grader expects. |
| Can't open Codespace | Try a different browser, or close and reopen from your repo |
| Lost work | Check git history: `git log --oneline` in the terminal |

---

## The Cell Pattern (Every Question Looks Like This)

```python
# Q3. Compute the posterior alpha for Variant A.
# Variant A: 1000 visitors, 40 conversions. Prior: Beta(2, 48).
q3_posterior_alpha_a = None     # ← replace None with your answer
print(f'Q3: {q3_posterior_alpha_a}')
```

→ Replace `None` with your answer:

```python
q3_posterior_alpha_a = 2 + 40   # prior alpha + conversions = 42
print(f'Q3: {q3_posterior_alpha_a}')
# Output: Q3: 42
```

---

## Codespace Tips — including how to come back to unfinished work

**You do not have to finish a homework in one sitting.**

- **Closing the browser is safe.** Your Codespace stops on its own after about 30
  minutes of inactivity, and everything you saved is still there next time.

- **Reopen the same one.** Your repo → **Code → Codespaces**, or
  github.com/codespaces, and click the Codespace you already have. Don't create
  a second one.

- ⚠️ **Push before a long break.** GitHub deletes a stopped Codespace after **30
  days without use, and it does that even if it holds work you never pushed.**
  Once your work is pushed, the Codespace is disposable — if it ever disappears,
  create a new one and everything is still on GitHub.

- **Stop it when you finish for the day** — a free account includes 120
  core-hours per month, which is **60 hours** on the default 2-core machine.
  github.com/codespaces → `...` next to your Codespace → **Stop codespace**.

- **Don't upgrade packages.** The versions are pinned so your results stay
  consistent with what the grader expects. If something seems broken, rebuild the
  container instead.

---

## Part A vs. Parts B and C

| | Part A | Parts B and C |
|---|---|---|
| Questions | Q1–Q10 | Q11–Q17 |
| Type | Math by hand | Agent + interpretation |
| Collaboration | Individual — no collaboration | Permitted |
| Autograded | ✅ Yes | ✅ Yes |
| How scored | Exact or tolerance match | Tolerance match (B) / Exact letter (C) |

**Your Part A numbers are your own.** The constants in Part A are generated
specifically for you, so your classmates' numbers are different and a copied
answer is wrong by construction.

---

## Getting Help

1. **Check the lecture notes first** — the worked examples match the homework structure
2. **Post to Canvas discussion** — classmates may have solved the same issue
3. **Office hours** — by appointment via Zoom — https://calendly.com/sh-lee-1/15min
4. **Email** — sh.lee@northeastern.edu — allow 48 hours

---

*The setup steps are done once. The submission steps are identical for every
assignment, hw01 through hw11.*
