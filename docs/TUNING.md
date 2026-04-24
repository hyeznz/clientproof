# 튜닝 가이드

조정 가능한 값들과 효과. "이거 좀 바꾸고 싶어"일 때 어디를 건드려야 하는지.

---

## 메인 타이틀 (3줄 히어로)

### 폰트 크기 스케일

HTML `fitMainTitle` IIFE 내부:
```js
const MAX_SIZE_RATIO = 2.15;
```

| 값 | 효과 |
|---|---|
| 1.5 | 짧은 줄이 상단과 비슷한 크기 (자간이 매우 넓어짐) |
| 1.8 | 짧은 줄 살짝 크고 자간 여유 있는 밸런스 |
| 2.15 | **현재값** — 자간 ~13%, 폰트 ~87% |
| 2.5 | 거의 폰트만으로 맞춤 (자간 거의 없음, 짧은 줄 압도적) |

### 자간/워드스페이싱

CSS `.mt-line--tight`:
```css
word-spacing: -0.1em;    /* 음수 = 단어 사이 좁게 */
padding-left: 0.04em;    /* 짧은 줄 오른쪽 시프트 */
```

- `word-spacing`: -0.05em~-0.2em 범위. 값 낮출수록 단어 간격 좁아짐
- `padding-left`: 0~0.08em. 크면 흔치 못 오른쪽으로 쏠림

### 화면 폭에 따른 기본 사이즈

```css
font-size: clamp(22px, 8.6vw, 66px);
```

- 첫 인수 (22px): 최소. 아주 좁은 화면에서의 크기
- 중간 (8.6vw): 뷰포트 비례. 숫자 높이면 더 큼
- 마지막 (66px): 상한. 데스크탑에서 최대 크기

---

## 카드 제목 (.cell-keyword)

### 폰트 크기 (CSS clamp)

```css
.cell-keyword {
  font-size: clamp(14px, 6.8vw, 26px);  /* 모바일 */
}
@media (min-width: 768px) {
  .cell-keyword { font-size: 36px; }  /* PC */
}
```

- 모바일 26px 기준. "v04w10 감성" 유지하려면 이 값 고수.
- `6.8vw` → 뷰포트 비례. 좁은 화면에서 자연 축소.
- PC 36px 고정. body max-width 720px 내에서 카드 폭 일정.

### 줄 간격 (line-height)

```css
line-height: 1.2;  /* 모바일 */
line-height: 1.18; /* PC */
```

1.0~1.3 범위. 낮을수록 빡빡, 높을수록 여유. 
1.0 미만은 어센더 클리핑 위험.

### 자간 (letter-spacing)

```css
letter-spacing: -0.03em;
```

-0.01em~-0.05em 범위. 음수로 살짝 조이는 느낌. 
점프체(KBL Jump)는 각진 서체라 -0.03em이 적절.

### 카드 내부 패딩

CSS `:root`:
```css
--pad: clamp(9px, 4.8vw, 18px);
```

- 9px: 좁은 화면에서의 최소 (카드 내부 여백 너무 빡빡하지 않게)
- 4.8vw: 뷰포트 비례
- 18px: 일반 모바일 상한

PC에서는 media query로 28px 고정.

---

## 벤토 컬러 배치 (scoreArrangement)

### 각 룰 가중치

HTML `scoreArrangement()` 내부:

```js
// Rule A: 같은 non-light 색 인접
if (arr[i].v !== 'light' && arr[i-1].v === arr[i].v) penalty += 80;   // 거리 1
if (arr[i].v !== 'light' && arr[i-2].v === arr[i].v) penalty += 35;   // 거리 2
if (arr[i].v !== 'light' && arr[i-3].v === arr[i].v) penalty += 12;   // 거리 3
if (arr[i].v !== 'light' && arr[i-4].v === arr[i].v) penalty += 4;    // 거리 4

// Rule B: 좌우 쌍 높이 같음
penalty += 8;

// Rule C: 큰 카드(l/xl) 연속
penalty += 18;

// Rule D: light 4+ 연속
penalty += 3;

// Rule E: 강조색 간격 std
penalty += Math.round(std * 3);

// Rule F: 짝수/홀수 강조색 불균형
penalty += Math.abs(evenAccents - oddAccents) * 4;

// Rule G: orange 첫 4장 안
if (firstOrange > 3) penalty += (firstOrange - 3) * 30;
```

### 검색 반복 횟수

```js
for (let t = 0; t < 300 && bestScore > 0; t++) {
```

300회 기본. 더 늘리면 더 좋은 배치 확률↑ (비용: 탭 로드 시간 살짝 증가).

---

## 후기 재배정 (rearrangeForFit)

### 목표 폰트 크기 (minSize 분류 기준)

HTML `getArrangementForFilter()`:
```js
const isDesktop = window.innerWidth >= 768;
const targetFS = isDesktop ? 36 : 26;
```

이 값 기준으로 "어느 사이즈 카드면 몇 줄 가능" 판정.

### SAFETY 여유

구버전(v04w22~23)에 있던 상수:
```js
const SAFETY_W = 0.96;   // 가로 96%
const SAFETY_H = 0.98;   // 세로 98%
```

현재(v04w24+)는 fitCardTitles가 noop이라 SAFETY 의미 없음. 
부활시킬 경우 0.90~0.98 범위.

---

## 후기 마크다운 자동 교정 스크립트

### fix_brands_and_breaks.py

```python
MIN_LINE_LEN = 6     # 최소 줄 길이
SOFT_TARGET = 18     # 선호 줄 길이 (쉼표 브레이크)
HARD_MAX = 28        # 강제 분할 임계값
```

- `MIN_LINE_LEN` 6 미만은 거의 안 건드림
- `SOFT_TARGET` 18자 이상이면 쉼표에서 브레이크
- `HARD_MAX` 28자 초과 줄은 자연 지점에서 강제 분할

값 올리면 **덜 자주** 줄바꿈 (더 긴 줄), 낮추면 **더 자주**.

### fix_particles.py

패턴 리스트 `SUBS` 확장 가능. 새로운 어색한 조사 발견 시 추가.

---

## 바디 max-width (데스크탑 중앙 영역)

CSS `:root` 또는 body 관련:
```css
@media (min-width: 768px) {
  body {
    max-width: 720px;
    margin-left: auto;
    margin-right: auto;
  }
}
```

- 720px: "데스크탑에서 카드 그리드 폭" 결정
- 높이면 카드가 더 넓어지고 폰트가 시각적으로 덜 큼
- 낮추면 더 좁아지고 모바일 느낌 강함

---

## 색상 값

CSS `:root`:
```css
--light-gray: ...;
--dark-gray: ...;
--orange: ...;
--black: ...;
```

브랜드 컬러 변경 시 여기서 한 번에.

### v-blue (체크용 마커)

```css
.cell.v-blue { background: #2563eb; color: #ffffff; }
```

파란색 다른 톤으로 바꾸고 싶으면 hex 코드만 교체.

---

## 애니메이션 timing

검색 키워드: `transition`, `animation`, `stagger`

- 카드 fade-in stagger: `--card-index`로 순차 애니메이션
- chip transition: `.18s ease`, `.12s ease` 등
- 시트 슬라이드: CSS transition

각 위치에서 duration/easing 조정 가능.
