# 브랜치 워크플로우

## 브랜치 구조 (하위 → 상위 순서)

```
feat/*, fix/*  ──PR──▶  dev  ──PR──▶  staging  ──PR──▶  main
```

- **main**: 최종 릴리스 브랜치. 항상 배포 가능한 상태만 유지.
- **staging**: main에 올리기 전 최종 검증 브랜치.
- **dev**: 모든 기능(feat)/수정(fix) 브랜치가 **먼저 합쳐지는** 통합 브랜치.

## 규칙

1. 모든 `feat/*`, `fix/*` 브랜치는 **dev**에서 분기한다.
   ```bash
   git checkout dev && git pull
   git checkout -b feat/기능명
   ```
2. 작업 완료 후 PR의 base는 **dev**. (`feat/* → dev`)
3. dev에 머지된 기능이 검증되면 **staging으로 승격 PR**. (`dev → staging`)
4. staging 검증 후 **main으로 승격 PR**. (`staging → main`)
5. 하위 브랜치로 직접 머지하지 않는다 (예: feat → main 금지).
6. 머지는 squash가 아닌 일반 merge 권장 — 승격 이력을 브랜치 그래프로 추적 가능해야 함.

## 승격 체크리스트 (각 PR마다)

- [ ] 이전 단계 브랜치에서 문제없이 동작 확인
- [ ] 컨플릭트 없거나 해결됨
- [ ] (staging→main) 릴리스 노트/버전 태그 예정 여부
