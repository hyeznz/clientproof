# 아키텍처 가이드

HTML 파일 하나로 구성된 단일 페이지 앱. 빌드 도구 없이 브라우저에서 바로 실행.

---

## 페이지 구조 (위에서 아래로)

```
<body>
├─ .topline-wrap        상단 구분선
├─ .title-block         메인 타이틀 (3줄 히어로 카피)
│   └─ h1.main-title
│       └─ span.mt-line × 3
├─ .banner              오렌지 배너 (후기 진정성 안내)
│   └─ .banner-inner
│       └─ .banner-body
├─ .filter-bar          언더라인 탭 (가로 스크롤)
│   └─ .chip × 10 (하이라이트/유형별이사/실력의차이/...)
├─ .bento-section       후기 카드 벤토 그리드
│   └─ .bento
│       └─ .cell × N    (카드 하나당)
│           ├─ .cell-top (상단 라벨 + 화살표)
│           └─ .cell-keyword (좌하단 제목, 3-4줄)
├─ .bottomline          하단 구분선
├─ .footer-info
└─ aside.sheet          카드 클릭 시 뜨는 상세 시트
```

---

## CSS 커스텀 프로퍼티

```css
:root {
  --row-unit: 40px;       /* 벤토 행 단위 (PC: 60px) */
  --gap: 8px;             /* 카드 사이 간격 (PC: 14px) */
  --radius: 14px;         /* 카드 모서리 */
  --pad: clamp(9px, 4.8vw, 18px);  /* 카드 내부 여백, 반응형 */
  --edge: calc(16px + env(safe-area-inset-left));  /* 화면 좌우 여백 */
}
```

PC(≥768px)에서 media query로 `--row-unit: 60px`, `--pad: 28px`, `--edge: 28px`로 바뀜.

---

## 색상 변형 (v-light, v-dark, v-orange, v-black, v-blue)

각 카드는 `v-{색}` 클래스로 4가지 색상 변형. `v-blue`는 체크용(다른 탭 중복 후기 마커).

```css
.cell.v-light  { background: var(--light-gray); color: var(--dark-gray); }
.cell.v-dark   { background: var(--dark-gray); color: var(--light-gray); }
.cell.v-orange { background: var(--orange); color: var(--black); }
.cell.v-black  { background: var(--black); color: var(--orange); }
.cell.v-blue   { background: #2563eb; color: #ffffff; }  /* 체크용 마커 */
```

카드 높이 변형: `h-xs` (3행), `h-s` (4행), `h-m` (5행), `h-l` (7행), `h-xl` (9행).

---

## 주요 JavaScript 모듈

### 1. 메인 타이틀 blockFit (`fitMainTitle` IIFE)

페이지 최상단 타이틀 3줄을 배너 폭에 맞춰 동적 사이징.

```
모든 줄 100px로 측정 → baseFS 산출 → MAX_SIZE_RATIO 캡 
→ 캡된 줄은 letter-spacing으로 나머지 채움 (반복 피팅)
```

주요 상수:
- `MAX_SIZE_RATIO = 2.15` — 짧은 줄 폰트 최대 배율
- 결과: 모든 줄이 배너 폭과 일치, 자간/폰트 하이브리드 분배

### 2. 후기 재배정 (`rearrangeForFit`)

목표: 긴 제목 후기는 큰 카드에, 짧은 제목은 작은 카드에.

```
filtered(내러티브 순서) + pattern(벤토 사이즈 배열)
  ↓
각 후기의 minSize 분류 (classifyMinSize)
  ↓
DESC zip: 포지션 사이즈 내림차순 ↔ 후기 요구 내림차순
  ↓
promote: 아직 안 맞는 포지션은 사이즈 승격
```

반환: `{ arranged: 후기배열, pattern: 수정된 벤토패턴 }`

### 3. 카드 사이즈별 dim 계산 (`computeCardDims`)

CSS 변수 + 샘플 카드 실측으로 `availW`, `availH[size]` 반환.

**주의**: `--pad`가 `clamp()` 식이라 `getPropertyValue`로 resolve 안 됨 
→ 샘플 카드의 computed padding을 실측해야 정확.

### 4. 벤토 패턴 생성 (`getPatternForFilter`)

- `hashSeed(filterName)`로 필터별 고유 시드
- `seededShuffle(bentoPool, rng)`을 300회 반복, 최저 점수 배치 선택
- 점수: `scoreArrangement()` 참조

### 5. 배치 점수 (`scoreArrangement`) — 벤토 미학

7가지 룰의 가중치 합산:

| 룰 | 페널티 |
|---|---|
| A. 같은 non-light 색 인접 (거리 1~4) | 80 / 35 / 12 / 4 |
| B. 좌우 쌍 같은 높이 | 8 |
| C. 큰 카드(l/xl) 연속 | 18 |
| D. light 4+ 연속 | 3 |
| E. 강조색 간격 표준편차 | std × 3 |
| F. 짝수/홀수 인덱스 강조색 불균형 | abs 차이 × 4 |
| G. Orange 첫 4장 안 노출 (above the fold) | (idx - 3) × 30 |

### 6. 카드 제목 피팅 (`fitCardTitles`)

현재는 **noop** 수준 — `inline font-size`만 리셋. 실제 FS는 CSS clamp이 담당.

(이전 버전들에선 JS가 전역 minFS 계산해서 덮어썼지만, 
"v04w10 사이즈가 제일 좋았다" 피드백 후 CSS 기반으로 환원함.)

### 7. 렌더 (`render`)

```
activeType → getArrangementForFilter → {arranged, pattern}
  ↓
grid.innerHTML = arranged.map(후기 → <article>).join('')
  ↓
fitCardTitles() → CSS 기본값 주입 (inline 제거)
```

렌더 내부에서 중복 후기는 `v-blue`로 오버라이드 (유형별이사/실력의차이 외 탭).

---

## 주요 데이터 구조

### reviews 배열 (~line 1011)
```js
const reviews = [
  { id:85, source:"best", nickname:"...", date:"...", 
    type:"일반포장", mainTag:"하이라이트", 
    tags:["하이라이트","친절응대"], newTitle:"..." },
  ...
];
```

### titleBreaks 매핑 (~line 1167)
```js
const titleBreaks = {
  85: '인생 최악의\n이사,\n여기서 끝',
  ...
};
```
ID별 카드 제목 고정 줄바꿈. `\n`이 줄 구분자.

### fullTextMap 매핑 (~line 1098)
```js
const fullTextMap = {
  85: "10월 15일 이사였는데,\n이것저것 정리할 게...",
  ...
};
```
카드 클릭 시 시트에 뜨는 본문 전문. 문단 구분은 `\n\n`.

### bentoPool 배열 (~line 1289)
20개 엔트리의 높이/색상 분포. 필터마다 seeded shuffle로 배치.

```js
// xs(3) + s(6) + m(6) + l(3) + xl(2) = 20
// light 11 + dark 4 + orange 3 + black 2 = 20
```

### typeLabel
```js
const typeLabel = {
  '일반포장': '일반 포장이사',
  '보관이사': '보관이사',
  // ...
};
```
이사 유형 → 카드 상단 라벨 변환.

---

## 데이터 플로우 (탭 클릭 → 화면 업데이트)

```
chip.click → activeType 변경 → render()
  ↓
getArrangementForFilter(activeType) [캐시 조회]
  ├─ 캐시 없으면: filter + sort + pattern + rearrange + promote
  └─ 캐시 있으면: 즉시 반환
  ↓
arranged.map → HTML 문자열
  ↓
grid.innerHTML = HTML
  ↓
fitCardTitles() [CSS 클램프 적용 보장]
```

**캐시 무효화**: window resize 또는 `document.fonts.ready` 시 
`_arrangementByFilter = {}` 하고 re-render.
