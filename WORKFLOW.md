# 브랜치 워크플로우 (A: cherry-pick 승격)

## 구조

```
main (=prod 서버)
 ▲  cherry-pick 승격 PR
staging (=staging 서버)
 ▲  cherry-pick 승격 PR
dev (=통합 테스트 서버)
 ▲  squash-merge PR
feat/*, fix/*   ← 분기는 항상 main 기준
```

## 규칙

1. **분기**: `git checkout main && git pull && git checkout -b feat/기능명`
2. **dev 통합**: 기능 완료 → `feat/* → dev` PR (squash-merge 허용).
   dev에는 항상 모든 기능이 들어간다. 통합/회귀 테스트는 dev 서버에서.
3. **staging 승격**: dev에서 검증된 기능을 **올리기로 결정된 시점**에
   dev의 머지 커밋을 `staging`으로 cherry-pick (PR).
4. **main 출시**: staging 검증 통과 → 같은 커밋을 `main`으로 cherry-pick (PR).
5. cherry-pick는 항상 아래 코드 형태 커밋만 대상으로 한다 — 일반 머지 커밋 금지:
   - dev: squash 머지 1커밋 = 기능 1개 → 그대로 cherry-pick 가능
   - 승격 PR에는 `cherry picked from commit <dev SHA>` trailer를 남겨 추적성 확보
6. 환경 브랜치(dev↔staging↔main) 간 통상 머지 금지. 흐르는 건 cherry-pick뿐.
7. 충돌은 승격 시점에 해당 환경 PR에서 해결. 해결이 복잡하면 기능 소유자가
   dev로 되돌아가 rebase 후 다시 승격 시도.

## 기능별 적용 시점

staging/main 히스토리가 곧 "언제 어떤 기능이 그 환경에 들어갔나"의 대장이다.
예: A는 staging에만, B는 staging+main에 — 모두 각자 다른 시점 cherry-pick으로 표현.
