# 작업 매뉴얼

자주 하는 작업 패턴. 단계별 실행 가능한 수준으로 기록.

---

## 1. 새 버전 만들기 (파일 수정 시 항상 먼저)

```bash
cd C:\Users\Alon\Documents\Claude\Projects\ClientProof
# 최신 버전 확인
ls reviews_v04w*.html

# 새 버전 복사 (예: v04w27 → v04w28)
cp reviews_v04w27.html reviews_v04w28.html

# 루트에 HTML이 3개 이상 쌓이면 가장 오래된 버전을 아카이브로 이동
# (루트는 항상 최신 2개만 유지)
mv reviews_v04w26.html archive/v04_history/

# 이후 v04w28만 수정, v04w27은 그대로 보존
```

**원칙**:
- 기존 HTML을 직접 수정하지 말 것. 항상 복사 후 새 버전에서 작업.
- **루트에는 최신 2개 버전만 유지**. 새 버전을 만들 때마다 가장 오래된 `v04w*.html`은
  `archive/v04_history/`로 이동. (Claude는 이 규칙을 자동 적용 — `CLAUDE.md` 참조)
- 프로토타입(v01~v03)은 `archive/prototypes/`에 보관, 건드리지 않음.

---

## 2. 카드 제목 줄바꿈 수정

특정 후기의 카드 제목 줄바꿈이 어색할 때.

1. HTML 파일에서 `titleBreaks` 찾기 (대략 1170행 근처)
2. 해당 후기 ID의 값 수정:

```js
// BEFORE
73: '20년 만의\n이사,\n역시 전문가',

// AFTER (예: 2줄로)
73: '20년 만의 이사,\n역시 전문가',
```

**주의**:
- `\n`이 줄 구분. 최대 4줄 권장.
- 한 줄에 7자 이상이면 좁은 모바일에서 2차 wrap 가능성
- 4줄 제목은 m+ 카드에 배정되므로 재배정 자동 반영

---

## 3. 배너 카피 수정

1. HTML 파일에서 `<div class="banner-body">` 찾기 (대략 820행)
2. 내부 텍스트 수정. `<br>`로 줄 구분, `<strong>`으로 강조.

```html
<div class="banner-body">
  아래의 후기들은 <strong>실제 이사를 마친 고객님들께서 직접 남긴 이야기</strong>입니다.<br>
  칭찬 일색이라 어색하게 느껴지실 수 있습니다. 저희도 압니다.<br>
  ...
</div>
```

---

## 4. 컬러 배치 점수 조정

벤토 그리드의 색상 분포가 어색할 때.

1. HTML에서 `scoreArrangement` 함수 찾기 (대략 1370행)
2. 각 룰의 가중치 숫자 조정

```js
// 같은 색 인접 (rule A) — 높이면 더 엄격하게 분산
if (arr[i].v !== 'light' && arr[i-1].v === arr[i].v) penalty += 80;  // 숫자 올림

// Orange 첫 4장 (rule G) — 숫자 높이면 더 강하게 앞쪽에 배치
if (firstOrange > 3) penalty += (firstOrange - 3) * 30;  // 30 → 50 등
```

3. 검색 반복 횟수도 조정 가능:
```js
for (let t = 0; t < 300 && bestScore > 0; t++) {  // 300 → 500 등
```

---

## 5. 메인 타이틀 blockFit 튜닝

흔치 않아요 줄이 너무 크거나 작게 느껴질 때.

HTML에서 `MAX_SIZE_RATIO` 찾기:
```js
const MAX_SIZE_RATIO = 2.15;  // 조정
```

- 값 **높이면**: 짧은 줄 폰트가 더 커짐, 자간 간격 좁아짐
- 값 **낮추면**: 짧은 줄 폰트가 작아짐, 자간이 넓어짐

---

## 6. 카드 제목 크기 조정

모바일 FS 바꾸고 싶을 때.

HTML `.cell-keyword` CSS에서:
```css
.cell-keyword {
  font-size: clamp(14px, 6.8vw, 26px);  /* 26px 부분 조정 */
}
@media (min-width: 768px) {
  .cell-keyword { font-size: 36px; }  /* PC 36px 조정 */
}
```

- `14px`: 극단적으로 좁을 때 최소 크기
- `6.8vw`: 뷰포트 비례 (숫자 크면 더 큼)
- `26px`: 상한

---

## 7. 마크다운 후기 파일 재생성

후기 원본(마크다운) 정제가 필요할 때.

```bash
# Linux 샌드박스에서
cd /sessions/ecstatic-sleepy-hopper/scripts

# 옵션 A: 문단 재구성 (빈 줄 추가)
python3 reformat_reviews.py

# 옵션 B: 상호명 + 줄바꿈 교정
python3 fix_brands_and_breaks.py

# 옵션 C: 조사 교정
python3 fix_particles.py

# 옵션 D: 구분선 앞뒤 빈 줄 보장
python3 fix_dividers.py
```

**순서 권장**: 상황에 따라 다르지만 일반적으로 reformat → brands → particles → dividers 순.

**주의**: 실행 전 백업 필수. 이미 `seperateReviewFiles_backup_v1/`, `_v2/`가 있지만 새 작업이면 `_v3/` 만들기.

---

## 8. 새 후기 추가

1. HTML 파일에서 `const reviews = [...]` 찾기 (대략 1011행)
2. 원하는 위치에 새 엔트리 추가:

```js
{ id: 86, source: "best", nickname: "새닉네임", date: "2025.12.01", 
  type: "일반포장", mainTag: "실력의차이", 
  tags: ["실력의차이", "속전속결"], 
  newTitle: "새로운 제목" },
```

3. `titleBreaks`에 ID별 카드 제목 줄바꿈 추가:
```js
86: '새로운\n제목',
```

4. `fullTextMap`에 본문 전문 추가 (카드 클릭 시 시트에 뜰 내용):
```js
86: "첫 문단 첫 문장\n첫 문단 둘째 문장...\n\n둘째 문단...",
```

---

## 9. 탭(필터) 추가/수정

1. `.filter-bar .type-chips`에서 chip 추가:
```html
<button class="chip" data-type="새탭명">
  <span class="chip-label">새탭명</span>
</button>
```

2. `mainTagOrder` 배열에 추가 (정렬 순서):
```js
const mainTagOrder = ['실력의차이', '가격', ..., '새탭명'];
```

3. 각 후기의 `tags` 배열에 "새탭명" 추가하면 자동 필터링

---

## 10. 캐시 수동 무효화 (개발 중 디버깅)

브라우저 DevTools 콘솔에서:
```js
_arrangementByFilter = {};
_globalCardFS = null;
render();
```

탭 전환/리사이즈 없이 재렌더 가능.

---

## 11. 버전 비교

두 버전이 뭐가 다른지 확인할 때:

```bash
# Linux 샌드박스
diff reviews_v04w26.html reviews_v04w27.html | head -100

# 또는 특정 함수만
grep -A 30 "function rearrangeForFit" reviews_v04w26.html > /tmp/old.txt
grep -A 30 "function rearrangeForFit" reviews_v04w27.html > /tmp/new.txt
diff /tmp/old.txt /tmp/new.txt
```
