---
name: test-and-prove
description: Prove that a test actually catches what it claims to guard by breaking the guard and watching the test fail. Use when writing or adding tests, judging coverage or test gaps, or when the user asks "is this tested?", "write tests for this", "add tests", "how's our coverage", "TDD", "why didn't the tests catch this?", or "this test is flaky". Korean triggers "테스트 짜줘", "테스트 붙여", "이거 테스트 있나", "커버리지 어때", "왜 테스트가 못 잡았지", "플레이크". Not for simply running an existing suite once.
---

# Test and Prove

**Do not trust a passing test. Break the guard it protects and prove that the test catches it.**

A green suite does not mean "the code is right". It only means "these tests did not fail". A test that guards nothing is green too.

## Principles

1. Diagnose by opening files. Matching file names is only a first guess.
2. Prefer real objects to mocks. Stub only at the external network boundary.
3. Failure branches and permission boundaries come before the happy path.
4. Add to an existing test file before you create a new one.
5. Coverage percentage is a by-product, not a goal.

## Steps

### 1. Diagnose: do not judge by file names

Pairing `payment.py` with `test_payment.py` is a **hypothesis**. Open the actual test files and check.

The most common misdiagnosis: **the tests for A live in B's test file.** By file name, A looks untested. In fact it is well covered. Report that misdiagnosis and you invent a problem that does not exist.

Do not ask "is there a test file?" Ask these:

- Are the failure branches covered (`except`/`rescue`/`catch`, `else`, validation failures, rejected state transitions)?
- Are the permission boundaries covered (requests from users who are not allowed)?
- Are the paths that write to the outside world, spend money, or delete things covered?

### 2. Decide what to write

| Priority | Target |
|---|---|
| 1 | Failure branches: an exception ends in the intended response, not a 500 |
| 2 | Permission boundaries: an unauthorized request does not change state |
| 3 | Irreversible transitions: a second call fails cleanly |
| 4 | Happy path: usually already there |

Rules:

- Every controller or request handler gets at least one integration test. A unit test does not replace it.
- If the service underneath already has its own tests, the handler test checks **only the handler's job**: parameters in, then the response, redirect, message, or "no state change". Do not re-test the service logic.
- Every mock is real behavior you no longer see. If you can reach the code with real objects and a real datastore, do that.
- Add the test to the file where that topic already lives. When a topic is scattered, the next person repeats the same misdiagnosis.

### 3. Prove: the core of this skill

After you write a test, **always** run this:

```
remove one guard → run only that test → it must fail → restore the guard
```

- If it does not fail, the test guards nothing. Rewrite it.
- Make the mutation where it cannot leak, and treat restoring as part of the same step. Restore with `git restore <file>`, or run the mutation in a throwaway copy of the repo. Leave no backup copies behind, inside the repo or outside it: they outlive the run, and the next person cannot tell which copy is real. Check that `git diff` is clean afterwards. A mutation left in the code is an incident.
- Break each guard on its own. If you break several at once, you cannot tell which test catches what.

**Trap: reading "nothing ran" as "passed".** If the test filter matches nothing, zero tests run and zero fail. Some runners print a line that looks like a pass, and some exit with code 0. So in a mutation run, **always read the run count.**

| Runner | Caught | Not caught: rewrite the test | Name filter matched nothing: not a pass |
|---|---|---|---|
| Minitest | `1 runs, 1 assertions, 1 failures` | `1 runs, 1 assertions, 0 failures` | `0 runs, 0 assertions` |
| pytest | `1 failed` | `1 passed` | `1 deselected` (`-k`), exit code 5 |
| Go | `--- FAIL` | `ok` | `ok ... [no tests to run]` (`-run`), **exit code 0** |
| Jest | `Tests: 1 failed` | `Tests: 1 passed` | `Tests: 1 skipped` (`-t`), **exit code 0** |

Go and Jest exit with 0 here. A script or an agent that checks only the exit code reads it as a pass.

The mutation can show up as an error instead of a failure. That still counts as caught. But find out **why** it is an error. A second line of defense, such as a datastore constraint, may have caught it instead. That is worth knowing in itself.

### 4. Run

- Run **everything** CI runs. Do not run part of it and say "all passed". Many stacks run browser or system tests with a separate command.
- If a hook or sandbox blocks the local run, run it in an approved remote environment. Its location belongs in the repo's `AGENTS.md`, `CLAUDE.md`, or your notes. A block is not a reason to skip verification.
- **On a shared runtime you are not the only user.** Before you start, check the load and other people's jobs:

  ```
  uptime                                   # load above the core count means it is already busy
  pgrep -fl "pytest|jest|rspec|rails|go test"  # are other sessions' tests running?
  ```

  If someone else's job is running, do not start a load experiment. Your extra load can turn their tests flaky. If you did add load, check for leftover processes afterwards and remove them.
- **Watch the run count.** If it differs from the last run, find out why. The code changed, the datastore state moved, or someone committed.

### 5. Pin down what never runs with text

The suite never executes some files: deploy scripts, service units, CI config. For a guard inside such a file, add a **contract test that asserts its text**. Otherwise, when someone deletes the guard, the suite stays quiet and the guard is gone.

```python
script = Path("bin/deploy").read_text()
assert "set -euo pipefail" in script
```

It is cheap and it catches regressions. Know its limit: it does not prove that the file runs.

### 6. Report

- **Never write that a check passed if you did not run it.**
- Give the numbers exactly as printed: `459 runs, 1875 assertions, 0 failures`.
- If you ran only part of the suite, say what you did not run.
- Include the mutation result. "I added tests" is weaker than "if this guard is removed, this test fails".

## Flaky tests

Do not cover an intermittent failure by re-running. At minimum, pin down these:

- **Reproduction rate**: M of N runs alone, and M of N runs in the full suite. Count them separately. A load-dependent flake does not reproduce alone.
- **Where it stops**: compare a passing run with a failing run. The assertion count or the failing line shows which assertion breaks.
- If you could not capture the failure message, **say so.** Mark any guess about the cause as a guess.

One common cause: an immediate read with no retry right after an assertion that waits (Capybara `assert_selector`, Playwright `expect(locator).toBeVisible()`). The first one waits and the second one does not, so they break one tick apart. The fix is to turn the immediate read into an assertion that retries.

## Anti-patterns

| Do not | Instead |
|---|---|
| Judge coverage by file names | Open the test files and check |
| Split interfaces or inject dependencies just for tests | Reach the code with real objects |
| Stop because it passed | Break the guard and check that the test catches it |
| Raise the coverage percentage | Cover failure branches and permission boundaries |
| Force one test file per controller | Join the file where the topic lives |
| Re-run a flaky test until it passes | Record the reproduction rate and where it stops |
| Run part of the suite and say "all passed" | Run everything CI runs |

## When not to use this skill

- You only need to run the existing suite once and read the result.
- The change is one obvious line. YAGNI applies to tests too.
- You need production verification, not tests. That belongs to post-deploy probes, monitoring, and rollback. For an internal tool with a handful of users, a production end-to-end suite may itself be overkill.

## Credits

The idea is mutation testing, done by hand on the test you just wrote. [Stryker](https://stryker-mutator.io/), [mutmut](https://github.com/boxed/mutmut), and [PIT](https://pitest.org/) automate it across a whole codebase. The rules came from corrections by the person this skill was first built for.
