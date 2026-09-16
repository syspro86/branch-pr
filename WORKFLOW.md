# 브랜치 워크플로우

## 브랜치 구조

```
main ──(분기)──▶ feat/*, fix/*
                    │
                    ├──PR──▶ dev       (통합 검증 전용, 승격 경로 아님)
                    ├──PR──▶ staging   (출시 전 검증)
                    └──PR──▶ main      (최종 출시)
```

- **main**: 배포 가능한 최종 상태. feat가 직접 머지되는 유일한 종점.
- **staging**: main 직전 검증 환경. feat가 머지된다.
- **dev**: 여러 feat를 함께 띄워 상호작용을 테스트하는 **모래상자**. dev는 언제든 main+feat들을 재조합해 재구성할 수 있고, dev에서 staging/main으로 코드가 흘러가지 **않는다**.

## 규칙

1. 기능 브랜치는 반드시 **main**에서 분기한다.
   ```bash
   git checkout main && git pull
   git checkout -b feat/기능명
   ```
2. 같은 feat 브랜치를 **순서대로 세 환경에 머지**한다. 각 단계는 개별 PR.
   - `feat/* → dev` : 다른 기능들과 함께 통합 테스트
   - `feat/* → staging` : dev 검증 통과 후 승격
   - `feat/* → main` : staging 검증 통과 후 출시
3. **dev를 feat 브랜치에 머지/rebase로 끌어오지 않는다.** dev의 코드(=남의 기능)가 feat 조상에 들어와
   "단독 승격"이 깨진다. (feat/theme-toggle가 dev 기반으로 rebase되어 staging에 편승한 사고 사례)
4. 충돌은 각 환경 머지 PR 지점에서 해결한다.
   - staging/main에 이미 들어간 코드와 충돌하면 → 그 환경 브랜치를 feat에 머지하지 말고,
     머지 커밋 자체에서 해결하거나 conflict-free rebase로 대응한다.
5. 환경 브랜치(dev/staging/main)끼리의 머지는 없다. 흐르는 방향은 항상 feat → 환경.
6. 머지方式是 일반 merge (squash 금지) — 같은 feat 커밋이 여러 환경을 지나는 것을
   히스토리 그래프로 추적할 수 있어야 함.

## 기능별 상태 매트릭스 (예시)

| 기능      | dev | staging | main |
|-----------|:---:|:-------:|:----:|
| theme-toggle | ✅ | ✅ | ⬜ |
| speed-control | ✅ | ✅ | ⬜ |
