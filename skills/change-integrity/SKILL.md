---
name: change-integrity
description: Check that only your own changes went up, that nothing is missing, and that the push actually landed when you commit, push, or open a PR or MR. Use when committing or pushing, especially when the repo is shared with other sessions, agents, or people, or when the working tree holds changes you did not make. Also use when the user says "commit this", "open a PR", "push it", "merge it", "why is this PR empty?", "someone else's changes came along", or "the working tree is dirty". Korean triggers "커밋해", "PR 올려", "MR 만들어", "푸시해", "머지해", "남의 변경이 딸려 왔어". Not for a one-file commit in a repo only you use.
---

# Change Integrity

**A successful push does not mean your change went up.**

Branch refs can move under you because of other work. Check the remote commit SHA, the diff from the merge base, and the real head and changes of the PR or MR together. What you are allowed to run still follows the user's instructions and your harness rules.

## Principles

1. The working tree may not be yours. Check before you start.
2. Do not use `git add -A` by default. Name the paths.
3. Before you commit, check the base branch and the range of commits to include. Compare again after the commit.
4. After you push, check the remote SHA and the diff the PR or MR will contain.
5. When a guard blocks you, do not bypass it. Separate your changes.

## Steps

### 1. Before you start: who else is in this tree?

```
git branch --show-current
git status --short
git worktree list
git log --oneline origin/<base>..HEAD
```

A modified file you did not create may be someone else's unfinished change. Do not guess its owner. Keep it.
Even if the tree is clean, another worktree or session may use it, so the limits below still apply.

- Even after you create a new branch, someone can commit to it in the meantime.
- If someone switches branches, your branch ref can move back.
- Your edits can be overwritten by someone else's `checkout` or `reset`.

**In a shared tree, get your own working tree with `git worktree`.** You never switch branches, so you never move another person's HEAD, index, or files.

```
git fetch origin
git worktree add <new-path> -b <new-branch> origin/<base>
# ... work ...
git -C <new-path> status --short --untracked-files=all
git worktree remove <new-path>
```

Fill in the real default branch and a new path. Clean up only after the work is merged or abandoned.
If uncommitted, untracked, or ignored files worth keeping remain, keep the worktree. If removal is refused, do not force it.

### 2. Staging: name the paths

```
git add <the paths you changed>
git diff --cached --stat
git diff --cached              # check the changes inside each file too
```

`git add -A` sweeps other people's unfinished work into your commit. With one or two files, typing the paths is faster and safer.

If one file mixes your changes with someone else's and you cannot separate them, **do not commit that file.**
Report the missing file and what it affects. Do not mark the work done while a needed change is missing.

### 3. Right after the commit: look at the parents

```
git log --oneline origin/<base>..HEAD
git show --stat --oneline HEAD
```

Compare the commits since the base branch with the range you planned. Report any approved earlier commits that are included. Commits you did not plan to include go to a new branch, as in step 4-b. Do not delete a commit only because it looks unfamiliar.

### 4. After the push: diff remote against remote

```
git fetch origin
git rev-parse HEAD
git rev-parse origin/<branch>
git log --oneline origin/<base>..origin/<branch>
git diff --stat origin/<base>...origin/<branch>
git diff origin/<base>...origin/<branch>
```

Check that the remote head is the SHA you just pushed. If more commits were added, check how they relate and what they contain.
Three dots (`...`) compare the work branch's changes since the common ancestor. A two-endpoint diff can make new files on the base branch look like deletions. Also compare the base and head SHAs from the PR or MR API with the real diff.

Three common results:

| What you see | What it means |
|---|---|
| Expected files, content, and SHA match | Normal, within the range you checked |
| The diff is empty | Already merged, identical content, or the wrong ref. Check commit ancestry and the PR or MR state |
| Unexpected files or commits | Check the base, the comparison method, and the commit range. Separate them if needed |

**4-b. If the range is mixed, rebuild it on a new branch.** Keep the original branch and worktree.

```
git worktree add <new-path> -b <new-branch> origin/<base>
git -C <new-path> cherry-pick <reviewed commit SHAs>
git -C <new-path> diff origin/<base>...HEAD
git -C <new-path> push -u origin <new-branch>
```

Resolve conflicts in the new worktree and verify again. If an existing PR or MR must be replaced, name the target and its state.
Ask the user to approve that specific action only when you must rewrite an existing remote branch. First pin the remote SHA you reviewed, then use `git push --force-with-lease=refs/heads/<branch>:<reviewed remote SHA> origin HEAD:refs/heads/<branch>`.
A bare `--force-with-lease` protects less when a background fetch updates the remote-tracking ref.
If the push is refused because the pinned SHA differs, do not retry automatically with the newest SHA. Review the new changes first.

To find a commit you think was lost, use `git reflog` and commit ancestry. Objects can survive without a ref, but nothing guarantees they stay, so put any commit you recover on its own branch.

### 5. When a guard blocks you: do not bypass it

When a repo guard, hook, or secret scanner blocks a commit, **the cause may not be your change.** If it caught someone else's work that was in the tree, do not use a bypass flag. Re-stage only your paths and pass the guard that way.

```
git restore --staged -- <path you staged by mistake>
git add <only your reviewed paths>
```

Use these commands only on the index of your own worktree. Do not unstage another worker's staged changes.
An explanation of why a bypass is needed is not an approval. Resolve guard errors and permission problems under the rules that own them.

### 6. Last check before merging

- Check that CI ran on the branch's **latest commit SHA**. If you rebuilt the branch, the earlier pipeline result belongs to different code.
- After the merge, fetch, then check the PR or MR state, the merge or squash SHA, and ancestry on the base branch.
  Compare `git merge-base --is-ancestor <actual merge or squash SHA> origin/<base>` with the changed files.
  A squash or rebase merge may not keep the original head SHA, so use the final SHA the provider returns.

## Anti-patterns

| Do not | Instead |
|---|---|
| Use `git add -A` by default | Name the paths |
| Treat a successful push as done | Check the remote SHA, the three-dot diff, and the real PR or MR changes |
| Stop because the PR or MR link appeared | Check the diff it contains |
| Switch branches in a shared tree | Use `git worktree` |
| Pass a guard with a bypass flag | Separate your changes and pass it |
| Force-rewrite an existing branch | Split onto a new branch; for an approved rewrite, use a lease pinned to the remote SHA |
| Commit someone else's unfinished work with yours | Leave it out and say why in the message |

## When not to use this skill

- You change one file and commit it in a repo only you use.
- The deploy itself. That belongs to a deploy skill.

## Evidence for the commands

When you edit these instructions, check these cases in a scratch repo. Do not reproduce failures in a user's repo.

| Case | Expected result |
|---|---|
| Another file is added on the base branch | The three-dot diff shows only the work branch's changes |
| Uncommitted files remain in a worktree | A plain `remove` is refused and the files are kept |
| The branch was squash-merged | Verify ancestry with the final squash SHA, not the original head |
| Someone commits to the remote and it is fetched | A lease pinned to the previously reviewed SHA refuses the push |

- [git diff](https://git-scm.com/docs/git-diff): two-endpoint comparison and comparison from the common ancestor.
- [git push](https://git-scm.com/docs/git-push): a lease with an explicit expected SHA, and how a background fetch affects it.
- [git worktree](https://git-scm.com/docs/git-worktree): refusal to remove a modified worktree, and forced removal.

## Credits

The rules came from corrections by the person this skill was first built for.
