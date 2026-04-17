# Student Quick-Start Card
## Marketing Analytics — Getting to Your First Submission

---

## Five Steps to Submit Homework

```
Step 1: Accept the assignment
         → Click the link posted to Canvas
         → Authorize GitHub Classroom
         → Click your name in the roster
         → Wait 30 seconds for your repo to be created

Step 2: Open your Codespace
         → Go to your new repo on GitHub
         → Green "Code" button → Codespaces tab
         → "Create codespace on main"
         → Wait ~90 seconds

Step 3: Complete the notebook
         → In the Explorer panel, open homework_0N_*.ipynb
         → Replace each None with your answer
         → Run Kernel → Restart & Run All Cells

Step 4: Push to GitHub
         In the terminal at the bottom:

         git add .
         git commit -m "completed hw01"
         git push

Step 5: Check your result
         → Go to your repo on GitHub
         → Click the "Actions" tab
         🟡 Yellow = running (wait ~2 minutes)
         🟢 Green = all tests passed
         🔴 Red = some tests failed → click to see which ones
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
| Autograder shows 🔴 | Click the red X → click "grade" job → expand the failed test |
| "Module not found" | Run `!pip install [module_name]` in a new code cell |
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

## Codespace Tips

- **Stop your Codespace when done** — you get 60 free hours/month.
  GitHub → Codespaces → click `...` next to your Codespace → Stop.

- **Your work is saved even after you stop.** Your files are stored on GitHub,
  not on your laptop. If you restart, everything is still there.

- **One Codespace per assignment.** Don't create a new one for each session —
  just reopen the existing one from the Codespaces tab.

- **The terminal is your friend.** `git status` shows what you've changed.
  `git log --oneline` shows your commit history.

---

## Part A vs. Parts B and C

| | Part A | Parts B and C |
|---|---|---|
| Questions | Q1–Q10 | Q11–Q17 |
| Type | Math by hand | Agent + interpretation |
| Collaboration | Individual — no collaboration | Permitted |
| Autograded | ✅ Yes | ✅ Yes |
| How scored | Exact or tolerance match | Tolerance match (B) / Exact letter (C) |

---

## Getting Help

1. **Check the lecture notes first** — the worked examples match the homework structure
2. **Post to Canvas discussion** — classmates may have solved the same issue
3. **Office hours** — [times and location]
4. **Email** — [address] — allow 48 hours

---

*This card covers HW01–HW10. The setup is identical for every assignment.*
