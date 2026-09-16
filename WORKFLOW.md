# 브랜치 워크플로우 (최종안: dev 통합 + release 브랜치 승격)

## 구조

```
main (=운영 서버)   ◀──PR(머지 1회)── promote/YYYYMMDD
staging (=스테이징)  ◀──PR(머지 1회)── promote/YYYYMMDD-staging
dev (=개발 서버)     ◀──PR(squash)── feat/*        ← 전 기능 통합 테스트
feat/*, fix/*        ──분기── main
```

## 흐름

```
1. git checkout main && git pull
   git checkout -b feat/기능명
2. feat → dev : PR squash 머지  (기능 = 1커밋 = 통합 단위)
3. staging 배포: git checkout -b promote/YYYYMMDD-staging staging
   (staging tip은 이전 release 브랜치를 staging에 머지한 결과.
   이미 머지된 dev S는 git이 patch-id로 제외 판정 — PR에 안 뜨면 정상)
4. 배포일: git checkout -b promote/YYYYMMDD main
   git cherry-pick -x <dev의 squash 커밋 SHA들>   # dev 머지 순서대로
   git push -u origin promote/YYYYMMDD
   → 첫 pick 후 PR 생성 (base: main), GitHub가 머지가능성 자동 검사
5. 대상 변경: reset + 재pick + push --force-with-lease  (이력 보존 안 함)
6. 배포 머지: --no-ff 머지 커밋 1건 = 배포 이벤트. 롤백 = revert -m 1
```

## 규칙

1. feat는 **main 기준 분기**만 (남의 조상 오염 금지 — dev 기반 rebase는 feat를 오염시킨다)
2. dev/staging은 **squash 머지만** — 기능당 커밋 1개가 승격 단위의 전제
3. promote 브랜치는 **main 기준 생성**. 승격 대상은 항상 **dev의 squash 커밋 SHA** (feat 원본 해시 아님)
4. cherry-pick 순서는 **dev 머지 순서를 지킨다** — 순서 뒤집히면 그 기능의 diff가 전제하는 컨텍스트가 없어 충돌한다 (S 커밋은 "머지 시점 기준 diff"이기 때문)
5. promote 브랜치는 머지 전 **수정 가능 (force-push 허용)** — release 대상 확정 과정의 이력은 git 밖(ticket/PR 코멘트)에 남긴다
6. promote → main 머지는 **merge commit (--no-ff)** — `git log --first-parent main`이 릴리스 이력이 되도록
7. 각 커밋 트레일러 `cherry picked from commit …`는 `-x` 옵션으로 자동 — 원본 추적이 안 끊기는 유일한 장치
8. 롤백: 해당 머지 커밋 `git revert -m 1 <SHA>` = 배포 취소 1건

## 추적 체인

```
feat 원본 ──squash──▶ dev S ──cherry-pick -x──▶ promote ──merge──▶ main
                     ↑승격 대상은 이 해시        (트레일러 연결)   (--no-ff 머지 = 배포 이벤트)
```

- `git log --grep <dev-S-SHA>` : 어느 release에 포함됐나
- `git log --first-parent main` : 릴리스 이벤트 목록
- 환경 매트릭스: `git branch --contains <SHA>` 로 dev/staging/main 각 환경 포함 여부 확인

## 자동화 (구현 예정)

- 라벨/커멘트 트리거 → promote 브랜치에 cherry-pick push → PR 생성/갱신
- PR base = main, 머지 버튼 = 승인된 change record의 실행
- ITSM 연동 시 change record ↔ PR 1:1 (PR 머지 이벤트로 상태 전이)
