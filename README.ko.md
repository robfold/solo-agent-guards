# solo-agent-guards

[English](README.md) · 한국어

코딩 에이전트가 끝나지 않은 일을 「다 됐다」고 말하지 못하게 막는 에이전트 스킬입니다. 뒤에서 검토해 줄 사람 없이 AI 코딩 에이전트를 혼자 쓰는 사람을 위해 만들었습니다.

![Claude Code 세션: 환불 모듈에 테스트를 요청하면, 에이전트가 테스트 3개를 쓰고, 가드를 하나씩 부숴 테스트마다 실패하는 것을 보고, 코드를 되돌린 뒤 git diff가 깨끗하다고 보고합니다.](assets/demo.gif)

<sub>이 플러그인을 켠 Claude Code에서 작은 예시 프로젝트로 녹화했습니다. 기다리는 시간은 자르고 오래 도는 구간은 빠르게 돌렸습니다.</sub>

- **[`test-and-prove`](skills/test-and-prove/SKILL.md): 테스트가 통과했다고 아직 증명된 것은 없습니다.** 에이전트는 테스트가 지키는 가드를 빼고, 그 테스트만 돌려 실패하는지 보고, 가드를 되돌립니다. 그래도 초록이면 그 테스트는 아무것도 지키지 않으므로 다시 씁니다. 에이전트는 exit code가 아니라 실행 건수를 읽습니다. `go test -run X`와 `jest -t X`는 아무것도 돌지 않아도 0으로 끝나기 때문입니다.
- **[`change-integrity`](skills/change-integrity/SKILL.md): push가 성공했다고 아직 증명된 것은 없습니다.** 에이전트는 원격 SHA, 기준 브랜치와의 세 점 diff, PR·MR에 실제로 담긴 변경을 확인합니다. 자기가 바꾼 경로만 스테이징하고, 다른 세션의 미완성 작업은 건드리지 않습니다.

## 이렇게 보고합니다

`test-and-prove`를 쓰면 에이전트는 초록 결과만이 아니라 가드를 부순 결과를 보고합니다. 위 녹화에서 나온 보고입니다(원문은 영어).

> | `billing/refunds.py`에서 바꾼 것 | 돌린 테스트 | 결과 |
> |---|---|---|
> | `if order.refunded:` → `if False:` | `test_refund_rejects_already_refunded_order` | 1 failed |
> | `if amount > order.total:` → `if False:` | `test_refund_rejects_amount_over_total_and_leaves_order_untouched` | 1 failed |
> | `>` → `>=` (전액 환불을 막는 경계값 실수) | `test_refund_marks_order_refunded` | 1 failed |
>
> 실행마다 실패가 1건씩 나왔습니다. 그러니 테스트가 건너뛰어진 것이 아니라 실제로 골라져서 실패했습니다. 코드를 되돌린 뒤 `git diff billing/`은 비어 있고 전체 스위트가 다시 통과합니다(4 passed).

`change-integrity`를 쓰면 올리려던 것이 아니라 실제로 올라간 것을 읽습니다.

> push는 성공했지만 `git diff --stat origin/main...origin/fix-refund`에 파일이 3개 보이고, 제가 바꾼 것은 2개입니다. `config/billing.yml`은 다른 세션의 미완성 작업입니다. `origin/main`에서 제 커밋만으로 브랜치를 다시 만들어 `fix-refund-2`로 push했습니다. 새 PR에는 파일 2개만 보입니다.

<sub>`test-and-prove` 보고는 녹화에서 옮겼습니다. `change-integrity` 보고는 이름을 지어낸 예시입니다.</sub>

## 설치

Claude Code:

```
/plugin marketplace add robfold/solo-agent-guards
/plugin install solo-agent-guards@solo-agent-guards
```

다른 에이전트는 [skills CLI](https://skills.sh)로 설치합니다.

```
npx skills add robfold/solo-agent-guards
```

또는 [`skills/`](skills/) 안의 폴더를 에이전트의 스킬 폴더에 복사합니다. Claude Code는 `~/.claude/skills/`입니다.

## 쓰는 법

요청이 맞으면 스킬이 알아서 켜집니다. `test-and-prove`는 테스트·커버리지·플레이크 수정을 요청할 때 켜집니다. `change-integrity`는 커밋·push·PR·MR을 요청할 때 켜집니다. 이름으로 부를 수도 있습니다. 예: "환불 검사 테스트를 짜고 증명까지 해".

`test-and-prove`는 실행기가 찍은 숫자 그대로와, 새 테스트마다 가드를 부순 결과로 끝납니다. `change-integrity`는 원격 SHA와 PR·MR에 실제로 담긴 diff로 끝납니다.

## 다른 스킬이 맞을 때

- **코드베이스 전체의 변이 테스트:** [Stryker](https://stryker-mutator.io/), [mutmut](https://github.com/boxed/mutmut), [PIT](https://pitest.org/)를 쓰세요. `test-and-prove`는 같은 일을 방금 쓴 테스트 하나에 손으로 합니다.
- **테스트를 먼저 쓰기:** obra의 superpowers에 있는 [`test-driven-development`](https://github.com/obra/superpowers/tree/main/skills/test-driven-development)를 쓰세요. TDD는 코드가 생기기 전에 테스트가 실패하는 것을 봅니다. `test-and-prove`는 코드가 생긴 뒤에 가드를 부숩니다. 그래서 오래된 코드에 붙이는 테스트에도 쓸 수 있습니다.
- **성공이라고 말하기 전에 증거부터:** obra의 superpowers에 있는 [`verification-before-completion`](https://github.com/obra/superpowers/tree/main/skills/verification-before-completion)이 이 원칙을 넓게 다룹니다. 이 두 스킬은 그 말이 가장 자주 틀리는 두 곳, 테스트와 push를 더 깊게 다룹니다.
- **격리된 작업 공간 만들기:** obra의 superpowers에 있는 [`using-git-worktrees`](https://github.com/obra/superpowers/tree/main/skills/using-git-worktrees)를 쓰세요. `change-integrity`도 worktree를 쓰지만, 할 일은 실제로 무엇이 올라갔는지 확인하는 것입니다.

## 출처

규칙은 이 스킬의 첫 사용자가 에이전트에게 준 교정에서 나왔습니다. `test-and-prove`의 생각은 변이 테스트입니다. `change-integrity`는 명령 근거로 git 공식 문서를 링크합니다.

## 라이선스

[MIT](LICENSE)
