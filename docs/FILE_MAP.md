# 파일/폴더 지도

프로젝트 파일 위치와 역할. 경로는 Windows 기준.

---

## 프로젝트 루트

`C:\Users\Alon\Documents\Claude\Projects\ClientProof\`

### HTML 파일 (메인 산출물)

**루트에는 최신 2개 버전만 유지** (자동 아카이브 규칙 적용 — `CLAUDE.md` 참조)

- `index.html` — 초기 랜딩 페이지 (보조, 버전 관리 대상 아님)
- `reviews_v04w27.html` — **현재 최신 안정 버전** ← 작업 시 이 버전 참조
- `reviews_v04w26.html` — 바로 전 버전 (빠른 롤백/비교용, 루트 유지)

### 하위 폴더

- `CLAUDE.md` — **Claude 작업 지침서** (프로젝트 루트에 위치, 최우선 참조)
- `docs/` — 이 문서 폴더 (프로젝트 가이드)
- `archive/`
  - `archive/prototypes/` — v01~v03 초기 프로토타입 (구조가 다름, 건드리지 않음)
  - `archive/v04_history/` — v04 시리즈 이전 버전들 (새 버전 만들 때 자동 이동)
- `font/` — KBL 폰트 파일 (`.woff2`, 현재는 HTML에 base64 임베드)
- `font_original_pfb/` — 원본 폰트 백업
- `resources/` — 이미지 등 외부 리소스

---

## 후기 원본 파일

`C:\Users\Alon\Documents\DTP_Ideation\resources\`

### `seperateReviewFiles/` (작업 대상)

후기 원본(복구 버전) 마크다운. 8개 파일 × 총 78개 후기.

- `zmcReview_best_01_edit.md` — 10 리뷰
- `zmcReview_best_02_edit.md` — 10 리뷰
- `zmcReview_best_03_edit.md` — 10 리뷰
- `zmcReview_best_04_edit.md` — 12 리뷰
- `zmcReview_sub_01_edit.md` — 10 리뷰
- `zmcReview_sub_02_edit.md` — 8 리뷰
- `zmcReview_sub_03_edit.md` — 9 리뷰
- `zmcReview_sub_04_edit.md` — 9 리뷰

### 백업 폴더

- `seperateReviewFiles_backup_v1/` — 최초 원본 (모든 정제 전)
- `seperateReviewFiles_backup_v2/` — 문단 재구성 완료 후, 상호명/줄바꿈 작업 전

### 기타

- `RAW/` — 원천 데이터 (가공 전)
- `상호명교체 전 수정파일/` — 상호명 교체 전 한글 상태

---

## 스크립트 (작업 자동화)

`/sessions/ecstatic-sleepy-hopper/scripts/` (Linux 샌드박스 내)

Python 스크립트들. 재실행 가능.

- `reformat_reviews.py` — 마크다운 후기 문단 구조 재구성 
  (fullTextMap 기반으로 빈 줄 삽입)
- `fix_brands_and_breaks.py` — 상호명 교체 + 소프트 줄바꿈 적용
- `fix_particles.py` — 조사(을/는/가 등) 자연스럽게 교정
- `fix_dividers.py` — `---` 앞뒤 빈 줄 보장 (Setext H2 해석 방지)

---

## HTML 내부 주요 위치 (reviews_v04w27.html 기준)

| 대략 라인 | 내용 |
|---|---|
| 1-1000 | `<head>` — CSS, 폰트, 메타 |
| 78-96 | CSS 커스텀 프로퍼티 (`:root`) |
| 166-203 | `.title-block`, `.main-title`, `.mt-line` 스타일 |
| 189-246 | `.banner`, `.banner-inner`, `.banner-body` 스타일 |
| 366-500 | `.cell`, `.cell-top`, `.cell-keyword`, `.kw-line` 스타일 |
| 800-820 | `<body>` 시작, 히어로 타이틀 HTML |
| 820-850 | 배너 박스 HTML |
| 850-880 | 필터 탭 HTML |
| 880-920 | 벤토 그리드 마운트 포인트, 시트(카드 상세) |
| 1011-1080 | `const reviews = [...]` (후기 데이터) |
| 1098-1170 | `const fullTextMap = {...}` (후기 전문) |
| 1170-1280 | `const titleBreaks = {...}` (카드 제목 줄바꿈) |
| 1289-1315 | `const bentoPool = [...]` (벤토 패턴 소스) |
| 1370-1445 | `scoreArrangement()` 배치 점수 |
| 1426-1460 | `computeCardDims()` 카드 dim 계산 |
| 1472-1494 | `createRuler()`, `classifyMinSize()` |
| 1516-1560 | `rearrangeForFit()` DESC zip + promote |
| 1620-1710 | `getArrangementForFilter()`, `render()` |
| 1820+ | 시트(카드 상세) 로직, 탭 인디케이터 |
| 1900+ | 필터 탭 이벤트, 초기 렌더 호출 |

(행 번호는 버전마다 약간 달라질 수 있음. 함수 이름으로 grep 권장.)

---

## 자주 찾는 코드 검색 팁

```bash
# HTML 파일 내 함수 찾기
grep -n "function getArrangementForFilter" reviews_v04w27.html
grep -n "scoreArrangement\|bentoPool\|titleBreaks" reviews_v04w27.html

# CSS 속성 찾기
grep -n "\.cell-keyword\|\.kw-line\|\.mt-line" reviews_v04w27.html

# 상수 찾기
grep -n "MAX_SIZE_RATIO\|SAFETY_W\|LH_CSS" reviews_v04w27.html
```
