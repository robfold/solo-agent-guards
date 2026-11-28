---
name: decision-ledger
description: Keep this project's decisions in a shared ledger file (DECISIONS.md) that people, other agents, and the next session all read. Use when the user decides or reverses something about the work — direction, scope, names, bans, defaults ("let's…", "from now on…", "drop X", "cancel that") — open it before you answer, even for a one-line decision; when the user says "we already decided this" or "like we agreed last time"; and before you start or resume work from a spec, plan, or issue. Decisions about the project go in this ledger, not your own memory; memory holds one person's preferences, the ledger holds what everyone working on the project must follow. Korean triggers "이렇게 하자", "앞으로는", "지난번에 정했잖아", "기획서대로 이어서 해". Not for a user's personal preferences (memory) or for writing up architecture rationale in depth (architecture-decisions).
---

# Decision Ledger

**Close the chat and the decision goes with it. Write each decision in the ledger, and start each task by reading the ledger.**

The next session does not know this conversation. Even when notes exist, nothing makes it read them to the end. This skill fixes two moments: write the decision when it is made, and read the ledger when work starts.

## Ledger or memory?

| It goes in the ledger | It goes in memory |
|---|---|
| A decision about this project's work: what to build, scope, names, bans, defaults | One person's preferences and habits: editor settings, tone, how they like reports |
| Anyone working on the project must follow it: people, other agents, the next session | Only matters when this one user talks to this one agent |
| It should be reviewable in the repository's history | It is fine if only this user's agent sees it |

When in doubt, it is a project decision. A memory entry is invisible to the teammate or the agent that picks up the work tomorrow.

## The ledger

- Location: `DECISIONS.md` at the root of the repository. If the project already has a place for decisions (an ADR folder, the issue tracker, a project doc), use that place.
- Decisions only. Progress notes and conversation summaries belong elsewhere.
- One decision per entry, newest on top.

```markdown
## 2026-03-14 Checkout takes cards only
- Source: "drop bank transfer, cards only" (user, chat)
- Scope: checkout page, refund notice
- Reverses: 2026-03-02 "cards and bank transfer"
- Reference: docs/checkout-plan.md section 2
```

## Steps

### 1. Write the decision in the same turn

When the user sets or reverses a direction, scope, name, ban, or default, add an entry to the ledger in the same turn you act on it.

- Quote the source word for word. If you need to interpret it, add a separate `Reading:` line.
- After the quote, say who decided: user, relayed (by whom), or agent choice. The account that wrote a record is not always the one who decided; an agent may post under the user's account.
- Write dates as `YYYY-MM-DD`. "Today" and "last week" change meaning in the next session.
- When a decision is reversed, keep the old entry and end it with `→ reversed (YYYY-MM-DD)`. Name the old entry in the new entry's `Reverses:` line.
- Find the other lines that relied on the reversed decision (spec acceptance criteria, to-do lists, user-facing copy) and mark them `(conflicts with YYYY-MM-DD decision)`. If fixing them needs another decision from the user, leave the wording as is.

Done when: the new entry has a date, source, decider, and scope, and every line that relied on the old decision is marked.

### 2. Read the ledger before you start or take over

- Read the whole ledger. Copy the entries that apply to this task, one line each, to the top of your working notes. If none apply, write "0 decisions apply".
- Do not rely on entries marked `→ reversed`.
- If work in progress has no ledger, build one first. Collect decisions, newest first, from issue comments, checkpoints, interview notes, and related docs, with a source on each entry. If the work has a spec, put the ledger section at the top of the spec.
- If the user says "we already decided this" and the ledger does not have it, find the source in earlier records (chat history, issue comments, commit messages) and add it. If you cannot find it, say where you looked and ask.

Done when: your working notes list the decisions that apply (including "0").

### 3. Work from the source, not a summary

- If there is a spec, plan, or instruction, read it to the end before starting. Summaries, your own earlier summaries, and one-line checkpoints are only for finding where the source is.
- When the source and a summary disagree, follow the source and fix the summary.
- If the source is long and you read only part of it, write how far you read ("read through section 3"). Make no call that depends on the unread part until you have read it.

Done when: every document you rely on is marked "read in full" or "read in part (range)".

### 4. When decisions conflict

- Within the same scope, follow the later explicit decision. A narrow exception applies only inside its scope.
- If both look alive and the ledger cannot settle it, show both sources side by side and ask the user once. Record the answer with step 1.
- If the ledger and the shipped result (the live page, deployed settings, a sales page) disagree, find the record that produced the result. If the result is older than the decision, mark it as a conflict and raise the fix separately. If you cannot find the decision that changed the result, say so and ask.

## Signals

| What you see | What it means |
|---|---|
| The user said "we already decided this" | Step 1 did not write it, or step 2 did not read it |
| Work built from a summary was rejected | A condition in the source was lost in the summary (step 3) |
| You asked the same question twice | The first answer is not in the ledger |
| The ledger reads like a progress log | Non-decisions slipped in; keep only decisions and move the rest to the progress record |
