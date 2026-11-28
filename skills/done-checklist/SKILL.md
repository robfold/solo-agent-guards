---
name: done-checklist
description: Write the finish line before starting, turn every correction into a checklist line, and run both before saying "done". Use when the user doubts a result or points out a miss ("did you check?", "is it done?", "did you review it?", "you missed it again") — open it before you fix anything; when the user hands over work and asks for a finish line or a self-check; and right before you report "done" or "complete". Korean triggers "이거 확인했어?", "다 됐어?", "또 빠졌네", "끝났으면 보고해". Not for re-reading the evidence behind a single status verb (claim-audit) or proving one test catches a bug (test-and-prove).
---

# Done Checklist

**Ask yourself before the user has to. Write "done" only after the finish line and the checklist have both run.**

When one person works with one agent, checking stays the person's job. The check runs only when someone asks "did you check this?" This skill moves that question to two places the agent reads on its own: the finish line written before the work, and a checklist that grows with every correction.

## Two records

- **Finish line:** what must be true for this one task to count as done. Written fresh for each task, in the task note or the issue body.
- **Checklist:** every kind of miss the user has pointed out in this repository, one line per kind, in `CHECKLIST.md` at the repository root. If the repository already has a review checklist or a PR template, use that instead. For work without a repository (documents, writing, operations), use the shared-rules part of the work's spec or project rules.

## Steps

### 1. Write the finish line before starting

- Write 2–5 conditions you can check by looking. Not "it works", but "a wrong password shows an error message on the login screen".
- Give each condition its check: a command, a screen, or a file.
- Copy conditions the user gave word for word. Mark the ones you added with `(added)`.
- If the request is too vague to write conditions, ask about the single biggest gap before starting.

Done when: every condition has a check.

### 2. Turn each correction into a checklist line, in the same turn

When the user says "did you check this?", "you missed X", or "you didn't review it", add one line to the checklist in the same turn you fix the problem. Do the same when you find on your own that something you reported was wrong, and leave a correction wherever the wrong report lives (an issue comment, a document).

```markdown
- [ ] Check ad copy against the platform's ad policy before posting — quote the policy section (2026-03-14, after "did you run the policy check?")
```

- Write the kind, not the incident. Not "the March 14 banner", but "ad copy policy check".
- If the kind is already there, add the date to that line instead of a new line. On a second correction, add one line on why step 3 did not catch it.

Done when: each correction has one checklist line (new, or a date added).

### 3. Run both right before reporting

Before you write "done", "complete", or "finished", run every finish-line condition and every checklist line that applies to this task.

- Report a table: item | result (pass, fail, not checked) | evidence (command output, screen, file path).
- Use evidence from this turn. If you reuse an earlier result, mark it `earlier`.
- Write "done" only when everything passed. Otherwise report "m of n passed, not checked: …".
- If a check is a test, first prove the test catches the break (`test-and-prove`). If a check is a commit, push, or merge, confirm what is actually on the remote (`change-integrity`).

Done when: the report has the table, and the word "done" appears only when everything passed.

### 4. Use other eyes where yours miss

Mark conditions the maker tends to miss (wording, design, security) as `needs other eyes`. If a separate reviewer is available (a subagent, another session, a person), hand those to them. If not, write "self-review only" in the table.

## Signals

| What you see | What it means |
|---|---|
| The user asked "did you check?" | The checklist has no line for that kind, or step 3 was skipped |
| The same correction came twice | The line names an incident, not a kind, or step 3 picked too few lines |
| The user found a defect after "done" | A condition had no check, or old evidence was reported as new |
| The checklist is too long to run every time | Run only the lines that apply to this task, and write the rule you used above the table |
