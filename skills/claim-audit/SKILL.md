---
name: claim-audit
description: Check that every status claim rests on the kind of evidence its verb needs before you report it. Use right before reporting "merged", "deployed", "passed", "running in production", "sent", "fixed", or a result number (conversions, payments, signups); when the user asks "is that real?" or "did you actually check?"; and before you merge, deploy, or pick up work someone else built. Korean triggers "다 됐어?", "진짜 머지됐어?", "실적 몇 건이야", "운영에서 도는 거 맞아?". Not for choosing what "done" means for a task, or for proving a single test.
---

# Claim Audit

**A tool's output line is not evidence. Write a claim only after you re-read, from the target itself, the evidence its verb needs.**

Agents rarely get status wrong by lying. They pick their own evidence for "done", and nobody questions it. This skill checks the **kind** of evidence behind each claim. Whether a test really catches a break belongs to `test-and-prove`. What actually went up to git belongs to `change-integrity`.

## Verbs and evidence

| Claim | Evidence it needs | Look-alike that is not evidence |
|---|---|---|
| Merged | PR or MR state `merged`, and the merge SHA is an ancestor of the base | A "Merged!" line, an auto-merge that is only scheduled |
| Deployed | The version or commit marker the live URL serves | Exit code 0 from the deploy command, a green build |
| Passed | A pass from a check you have seen fail, plus the run count | `0 failures` with 0 runs, a check that is always true |
| N orders, N users (a result) | A number that went through a contamination check | A row count, a count of `status=paid` |
| Running in production | The last run recorded by the production scheduler | One run by hand, a local dry run |
| Sent | The provider's or the receiver's delivery record | A saved draft, a queued job |
| Fixed | The repro failed before the fix and the same repro passes after | A pass after the fix alone |

For a verb not in the table, ask the same question: "Is this output a record of what I did, or the current state of the target?"

## Steps

### 1. Pull out the claims

From your draft report, list every status verb (done, merged, running, passed) and every number. Include numbers in tables, summaries, and parentheses. Side numbers go wrong more often than the headline.

Done when: every status verb and number in the draft is on the list.

### 2. Close the four gaps for each claim

Ask each claim these four questions in order. For every gap that applies, re-read the target.

- **Scheduled gap**: did you see "done", or "request accepted"? Re-read the state of the target from its API, the remote, or the live URL, not from the command's output. If it is still pending, wait for it or report it as scheduled.
- **Checker gap**: have you ever seen the check behind this claim fail? If not, feed it one input that must fail and confirm that it fails. If the result is "all passed" or "0 found", look at one or two samples yourself.
- **Sample gap**: does the number include your own tests, smoke tests, mock payments, scripts, or internal accounts? Split it by flows that finished within the same second, mock or test flags, empty source fields (utm, channel), user agents such as curl, and internal IPs or accounts.
- **Environment gap**: is the place you checked the place you are making the claim about? A production scheduler's environment variables, working directory, and dedicated worktree differ from a run by hand. Describe production state from production's config and its last recorded run.

Done when: every claim on the list carries "re-read from the target (with what)" or "could not re-read (why)".

### 3. For work someone else built, read the review state first

Before you merge, deploy, or take over work, read the latest review state on its card, PR, or MR. If a rejection, a change request, or a hold is still open, stop and report it word for word. The reviewer's last verdict outranks the builder's "done" comment and the card's status field.

Done when: each piece of work you merge or take over carries "latest verdict: … (who, when)".

### 4. Hand hard-to-undo claims to a second pair of eyes

For claims that touch money, public releases, production data, or messages to people, have a reviewer who did not build it (a subagent, another session, a person) run the check again. Give the reviewer the artifact and how to check it, not your conclusion: a reviewer handed a conclusion tends to confirm it. If no reviewer is available, write "self-checked only" in the report.

Done when: each such claim carries "reviewer: …, verdict: pass or reject" or "self-checked only".

### 5. Use the verb your evidence supports

- Use only as much verb as the evidence reaches: "merge scheduled (CI still running)", "manual run ok, last production run not checked", "paid: 31 (contamination not checked)".
- Keep unchecked numbers out of conclusions, titles, and summary cells.
- If you already reported a wrong claim, post the correction where the claim lives (the issue comment, wiki, commit message, chat).
- If you hit the same kind of gap twice, add one line for that kind to the repo's review checklist or agent rules (`AGENTS.md`, `CLAUDE.md`).

Done when: every status verb in the report stays inside the evidence from step 2.

## Common signals

| What you see | What it means |
|---|---|
| The tool printed success but the target looks unchanged | Scheduled gap. Re-read the target's state |
| A check you just wrote passed everything on its first run | Checker gap. Feed it a failing input first |
| The first day's number is better than expected | Sample gap. Filter out your own tests first |
| It works by hand but production shows a warning, or the reverse | Environment gap. Read production's config and run log |
| The user asks "is that real?" | Step 2 was skipped, or a verb in step 5 went past the evidence |

## Credits

The rules came from corrections by the person this skill was first built for. Each gap is a real incident: a merge tool that prints "Merged!" for a scheduled merge, a new check that reported 360 of 360 passed because it compared every input with itself, smoke-test payments reported as customer payments, and a manual run reported as production state.
