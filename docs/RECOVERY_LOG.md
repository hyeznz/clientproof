# 후기 본문 복구 이력

CSV 원본(`resources/seperateReviewFiles/RAW_v01/강봉원_고객감동후기_모음 - 시트1 (1).csv`)을 진리의 원천으로 삼아, html `fullTextMap`과 `_edit.md` 파일들에 누락된 문장을 복원하는 작업의 진행 이력.

---

## 📐 원본 소스 피라미드 (중요)

```
RAW_v01/CSV (골드 스탠다드 — 진짜 원본)
    ↓ (편집으로 일부 누락 발생)
RAW_v02/*.md (원본에 가까운 마크다운)
    ↓ (선정/재구성 과정에서 추가 누락)
resources/seperateReviewFiles/zmcReview_*_edit.md (편집본)
    ↓ (html 변환 과정에서 또 축약)
reviews_v04wXX.html의 fullTextMap (최종)
```

**뒤로 갈수록 내용이 줄어들거나 일부 문장이 사라져 있음**. 복구 시에는 항상 CSV를 기준으로.

---

## 🎯 복구 상태 (2026-04-24 세션 기준)

### ✅ html fullTextMap 반영 완료 — 총 74건

| 단계 | 대상 | 건수 | 커밋 |
|---|---|---:|---|
| 1차 | 소규모 누락 (diff < 50자) | 39건 | `0ebae7e` |
| 2차 | 제니퍼 id 55 (최장편 1,811자) | 1건 | `0ebae7e` |
| 3차 | 중/대규모 (diff ≥ 50자) | 34건 | 아직 uncommitted |

**복원 총량**: 약 **+13,000자** (중/대규모에서 +12,562자 포함)

### ⚠️ md `_edit.md` 파일 동기화 — 미완

| 단계 | 상태 |
|---|---|
| 소규모 39건 | ✅ 동기화 완료 |
| 제니퍼 id 55 | ✅ best_03_edit.md 반영 |
| 중/대규모 34건 | ❌ **미동기화** (이번 세션 때 알론이 "md는 나중에" 결정) |

### 🚧 남은 이슈

1. **id 84 루시드쏘**: CSV #84의 제목이 2줄이라 파서가 닉네임을 잘못 잡음 (현재 CSV에서 닉네임이 "에서 할래요!!"로 파싱됨). `outputs/parse_csv_master.py` 수정 필요. 본문 자체는 이미 html에 있으니 누락 여부만 재확인 대상.

2. **md 동기화 34건**: 위 3차 복구분을 `_edit.md`에 반영 필요. `apply_md_v2.py` 재활용 가능 (그러나 OUT_JSON 경로를 `medium_large_output.json`으로 변경해야 함).

3. **다른 탭에서 확인**: 알론이 휴대폰에서 각 탭(유형별이사/실력의차이/고난도돌파/책임감/가격/친절응대/속전속결/정리정돈) 별 후기 몇 개 펼쳐보고 가독성/길이 최종 확인.

---

## 🛠️ 작업 도구 (모두 `outputs/`에 저장)

`C:\Users\Alon\AppData\Roaming\Claude\local-agent-mode-sessions\.../outputs/` (리눅스 샌드박스에선 `/sessions/*/mnt/outputs/`):

| 파일 | 역할 |
|---|---|
| `parse_and_diff.py` | md ↔ html fullTextMap 비교 (기본) |
| `parse_csv_master.py` | CSV 원본 파싱 → `csv_master.json` |
| `three_way_diff.py` | CSV + html + md 3-way 비교 → `3way_diff.json` |
| `apply_recovery.py` | 39건 소규모 html 반영 (정규식 버그 있음 — v2 권장) |
| `apply_md_v2.py` | md 블록 단위 안전 반영 (39건 md 동기화에 사용) |
| `apply_jennifer.py` | 제니퍼 id 55 단건 반영 |
| `apply_medium_large.py` | 34건 중/대규모 html 반영 |
| `csv_master.json` | CSV 파싱 결과 (84건) |
| `3way_diff.json` | 3-way diff 상세 결과 |
| `small_recovery_input.json` / `output.json` | 소규모 39건 입출력 |
| `medium_large_input.json` / `output.json` | 34건 입출력 |
| `3way_summary.txt` | 요약 리포트 |

---

## 📋 다음 세션 체크리스트

새 채팅에서 이어갈 때 순서:

1. **현재 버전 확인**: `ls reviews_v04w*.html` — `v04w28`이 최신이어야 함
2. **git 상태 확인**: `git status`, `git log --oneline -5`
3. **복구 결과 확인**: 휴대폰/브라우저에서 제니퍼 카드(실력의차이 탭 → "책장 순서도 그대로, 강아지 종이도 그대로") 등 열어서 현재 상태 체감
4. **남은 작업 선택**:
   - md 동기화 (34건)
   - id 84 재매칭
   - 다른 카드 디자인/텍스트 수정
5. **docs 먼저 읽기**: `DESIGN_RULES.md`, `PITFALLS.md`, `WORKFLOW.md`, 이 파일

---

## 🔗 주요 파일 경로

- **현재 최신 html**: `reviews_v04w28.html`
- **공개 페이지**: `index.html` (v04w28 내용으로 동기화됨)
- **원본 CSV**: `resources/seperateReviewFiles/RAW_v01/강봉원_고객감동후기_모음 - 시트1 (1).csv`
- **md 편집본 8개**: `resources/seperateReviewFiles/zmcReview_{best|sub}_0{1-4}_edit.md`
- **제니퍼 참고용 PDF**: `resources/cafeCapture/제니퍼.pdf`

---

## 🧠 왜 이 작업을 했는가?

- 초기 html 제작 시 모바일 가독성을 위해 많은 후기를 과하게 축약하거나 편집 과정에서 문장 누락이 있었음.
- 알론이 CSV 원본을 비교하다가 "초코바닐라 후기의 '음료수' 문장이 html에 없네?" 지적하면서 발견.
- 전수 조사 결과: 78건 중 76건에서 CSV 대비 html/md 누락 확인.
- 2026-04-24 세션에서 대부분 복구 완료. 남은 작업(md 34건 동기화 등)은 향후 세션.
