# ClientProof 프로젝트 가이드

이사짐캐리(구 GS이사몰) 고객 후기 검증 페이지를 담은 프로젝트. 
이 문서들은 새로운 Claude 세션에서 이 프로젝트 작업을 이어갈 때 
맥락을 빠르게 복원하기 위한 가이드 모음.

## 🔔 핵심 규칙 한눈에 보기 (새 세션 필수 인지)

아래 6가지는 어떤 작업을 하든 반드시 지켜야 하는 불문율.
각 규칙 뒤에 상세 출처를 명시했으니 이해 안 되면 해당 문서 섹션으로 가서 확인.

1. **CSV가 진짜 원본** — 후기 본문 복구 방향은 항상 `CSV → html → md`. 역방향 금지. (→ `DESIGN_DECISIONS.md` §13, `RECOVERY_LOG.md`)
2. **줄 길이 14~22자** — 한 줄 목표 14~22자, 쉼표로 이어지는 짧은 단위 2개는 한 줄에 합침. 마침표 뒤 새 줄. (→ `DESIGN_RULES.md` §6)
3. **상호명 "이사짐캐리" + 받침 없음 기준 조사** — "GS/지에스" → 이사짐캐리. 조사는 `를/는/가/였/와/로/라고` (받침용 `을/은/이/이었/과/으로/이라고` 사용 금지). (→ `DESIGN_RULES.md` §5)
4. **"강봉원"은 카페 이름 → 그대로 유지** — 상호명 아님. 치환 대상 아님. (→ `DESIGN_RULES.md` §5)
5. **md 동기화는 `apply_md_v2.py` 방식만** — 블록 단위 split + 재조립. v1의 `re.sub + DOTALL` 방식은 버그로 사용 금지. (→ `PITFALLS.md` §16, `WORKFLOW.md` §12)
6. **폰트 축소(0.8em) 기능 구현 안 함** — 제니퍼(1,811자) 실물 테스트로 불필요 확정. 대신 줄 합치기로 가독성 확보. (→ `DESIGN_RULES.md` §6 끝, `DESIGN_DECISIONS.md` §12)

---

## 이 폴더에 뭐가 있어?

- **[DESIGN_RULES.md](./DESIGN_RULES.md)** — 절대 깨지면 안 되는 디자인 공식들 (읽기 1순위)
- **[ARCHITECTURE.md](./ARCHITECTURE.md)** — HTML/CSS/JS 구조와 주요 모듈
- **[DESIGN_DECISIONS.md](./DESIGN_DECISIONS.md)** — 왜 이렇게 만들었는지 (선택의 이유)
- **[PITFALLS.md](./PITFALLS.md)** — 잘 안됐던 함정들 (같은 실수 반복 방지)
- **[FILE_MAP.md](./FILE_MAP.md)** — 파일/폴더 위치와 역할
- **[WORKFLOW.md](./WORKFLOW.md)** — 자주 하는 작업 매뉴얼
- **[TUNING.md](./TUNING.md)** — 조정 가능한 값들과 효과
- **[RECOVERY_LOG.md](./RECOVERY_LOG.md)** — 후기 본문 CSV 원본 복구 진행 이력 (v04w28 세션 이후 신규)

## 프로젝트 개요

- **목적**: 강봉원 카페에 이사짐캐리(구 GS이사몰) 후기를 모아 보여주는 
  모바일 우선 반응형 페이지. 투명성과 신뢰를 시각적으로 전달.
- **기술**: 정적 HTML + 바닐라 JavaScript + CSS. 빌드 도구 없음. 
  브라우저에서 `.html` 파일 열면 바로 동작.
- **데이터**: 후기 78개 (best 42 + sub 36). HTML 파일 내 `reviews` 배열에 하드코딩.
- **폰트**: Pretendard(본문) + KBL Jump(디스플레이). KBL Jump는 로컬 base64 임베드.

## 최신 버전

**현재 안정 버전: `reviews_v04w28.html`** (2026-04-24 업데이트)

- v04w28 하이라이트: 후기 본문 74건을 CSV 원본 기준으로 복구 완료 (+약 13,000자). 자세한 내역은 `RECOVERY_LOG.md` 참조
- `index.html`이 v04w28 내용과 동기화되어 있어 GitHub Pages 배포 대상
- 이전 버전은 같은 폴더(최신 2개) + `archive/v04_history/`(오래된 것들)에 보존. 롤백/비교 필요 시 그대로 사용 가능

## 사용자 (알론)에 대한 참고사항

- 코딩 초보 — 기술 용어 설명을 쉽게, 두괄식으로 작성해 주면 좋음.
- 반말/친한 톤 선호 (절친처럼).
- 여성. 남동생 언급 시 "남동생"(오빠 아님) 정확히 구분할 것.
- 설명이 길어지면 지치므로, 핵심 먼저 + 선택적 상세.

## 작업 세션 시작할 때

1. **`RECOVERY_LOG.md` 먼저 훑기** — 직전 세션 상태/남은 작업 파악 (v04w28 이후 필수)
2. `DESIGN_RULES.md` 훑기 — 공식이 깨지면 안 됨 (특히 §6 줄 길이 규칙 v2는 신규)
3. 최신 HTML 파일(`reviews_v04wXX.html` 최고 번호) 사용
4. 수정 시 새 버전 생성 (`cp v04wXX.html v04wYY.html`) + 아카이브 규칙은 `CLAUDE.md` 참조
5. 수정 후 바로 브라우저/휴대폰(GitHub Pages: https://hyeznz.github.io/clientproof/)에서 확인 가능

## 문서 업데이트 원칙

이 `docs/` 안의 문서들은 프로젝트의 "집단 기억". 
중요한 결정/규칙/함정이 새로 생기면 해당 문서에 추가해서 
다음 세션에서도 참조할 수 있도록 유지.
