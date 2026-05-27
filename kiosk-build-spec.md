# Mailroom Kiosk — Build Spec

> VS Code에서 이 문서를 보고 그대로 만들면 됩니다.
> 화면 UI 텍스트는 **영어(발표 영어)**, 스펙 설명은 한국어.

---

## 0. 한 줄 정의 & Strength

**제품:** 라커 없는 무인 mailroom에서, 거주자가 택배를 빠르게 찾도록 돕는 키오스크.
하나의 단말, 두 모드 — **Store a package**(택배기사 입고) / **Find my package**(거주자 수령).

**Strength (발표 핵심):**
- *"We don't organize the pile — we make it searchable."* (더미를 정리하지 않고, 검색 가능하게)
- 라커 없이 **바닥 구역선(zone)** 만 그으면 됨 → 칸·잠금·용량 한계 없음, 막 쌓기 OK, 라커의 몇십 분의 일 비용.
- 구역 크기는 **페인트라 공간/시즌마다 조절** (라커는 하드웨어라 고정).
- 입고 시 **주소를 정면 스캔(계산대처럼)** → 택배사 파편화와 무관(주소 텍스트는 모든 라벨에 있음).

---

## 1. 전역 디자인 스펙 (Global)

| 항목 | 값 |
|---|---|
| 테마 | **라이트** (흰/오프화이트 배경, 진한 텍스트) |
| 화면 비율 | **세로 키오스크** (예: 1080×1920, 9:16). 실제 단말처럼 |
| 모드 색 (반대 색) | **Store = 앰버/오렌지 계열**, **Find = 블루/틸 계열** (hex는 디자인 시스템에서) |
| 타이포 | 큰 위계. 헤드라인 매우 큼, 본문 충분히 큼(저시력 대비) |
| 터치 타겟 | **크게** (서서·한 손·짐 든 상태 / Fitts). 버튼 최소 높이 넉넉히 |
| 대비 | **고대비** 필수 (라이트 테마라 텍스트·버튼 명도차 확보) |
| 구역 표기 | **색 + 이름 병기** ("Zone B" + 색) — 색각이상·저시력 대비 |
| 접근성 | 휠체어 도달 높이 고려, 핵심 정보는 화면 중앙~하단, 작은 글씨/복잡한 제스처 금지 |
| 자동 리셋 | 완료/유휴 시 진입 화면으로 (공용·익명 단말, 개인정보 잔존 0) |

---

## 2. 화면 맵 / 플로우

```
[ENTRY] What would you like to do?
   ├── 🟠 Store a package  → [A1 Select zone] → [A2 Scan address] → [A3 Done] → ENTRY
   └── 🔵 Find my package  → [B1 Enter address] → [B2 Your zone] → (Print/Done) → ENTRY
```

- 모든 화면에서 상단에 건물명 placeholder, 좌상단 Back(진입 외).
- 클릭으로 화면 전환되게(state machine). 브라우저 storage 쓰지 말 것(메모리 변수로).

---

## 3. ENTRY (공통 진입)

**목적:** 워크업 사용자에게 0.5초 안에 역할 분기.

**요소 / 영어 copy:**
- 상단: `[ Building Name ]`
- 헤드라인: **"What would you like to do?"**
- 큰 버튼 2개 (세로 스택):
  - 🟠 **"Store a package"** / 보조: `for couriers`
  - 🔵 **"Find my package"** / 보조: `for residents`

**행동:** 둘 중 탭 → 해당 모드 첫 화면.
**근거(발표용):** 단일 진입 + 명확한 2분기(Hick), 색=역할 signifier, 미니멀.

---

## 4. STORE 모드 (택배기사 입고) — 🟠

### A1 — Select zone
**목적:** 적재할 구역을 기사가 *현장 판단으로* 선택(강제 배정 X → 라커와 차별).

**요소 / copy:**
- 헤드라인: **"Where are you placing them?"**
- 구역 카드 그리드: **Zone A / Zone B / Zone C / Zone D** (수는 공간 따라 조절)
  - 각 카드: 구역명 + 색 + 현재 적재량 힌트 `12 items`
- 큰 카드, 충분한 간격.

**행동:** 구역 탭 → A2.
**근거:** Fitts(큰 카드), 시스템 상태 가시성(적재량), 자율 배정.

### A2 — Scan address (연속, 계산대식)
**목적:** 송장 주소를 스캔해 **호수 ↔ 구역** 등록. 연속으로 "띡띡띡".

**요소 / copy:**
- 상단: 선택된 구역 칩 `Zone B`
- 헤드라인: **"Scan each label's address"**
- 중앙: **스캐너 뷰파인더** (큰 박스, 라이브 카메라 느낌 / placeholder)
- 우측 또는 하단: **방금 스캔된 목록** 실시간 누적
  - 예: `1204  ✓` / `0907  ✓` / `1511  ✓`
  - 카운터: **"7 scanned"**
- 못 읽었을 때: **"Couldn't read — re-scan"** 안내 (에러 복구)
- 하단 버튼: **"Done"**

**행동:** 라벨 한 장씩 비춤(데모에선 "Scan" 버튼 = 1건 추가) → 같은 Zone에 우르르 → Done → A3.
**근거:** 즉각 피드백(스캔→목록 추가), recognition(읽힌 호수 보임), 정면 스캔이라 OCR 신뢰(더미 위 카메라와 달리 가림 없음), 택배사 무관(주소 텍스트).

### A3 — Done
**목적:** 묶음 등록 마무리 + 리셋.

**요소 / copy:**
- 큰 체크(placeholder)
- **"7 packages registered to Zone B"**
- 보조: `Returning to start…` (3초 카운트다운) + 버튼 **"Scan more"** / **"Done"**

**행동:** 자동/탭으로 ENTRY 복귀.
**근거:** closure, 자동 리셋(다음 기사).

---

## 5. FIND 모드 (거주자 수령) — 🔵

### B1 — Enter address (동·호수 한 번에)
**목적:** 거주자가 100% 아는 정보(동·호수)로 식별. 현실 습관대로 동·호수 함께.

**요소 / copy:**
- 좌상단 Back
- 헤드라인: **"Enter your building and unit"**
- 입력 디스플레이: `[ 102 ]  -  [ 1204 ]` 형태 (동 칸 채우면 호 칸으로 자동 이동)
- **큰 숫자 키패드** (0–9, ⌫, 확인)
- 보조문구: **"No app or tracking number needed."**

**행동:** 동·호수 입력 → 확인 → B2.
**근거:** Fitts(큰 키), 숫자=언어·연령 무관(universal), 동·호수 함께=현실 습관(인지부담↓), 저민감 정보만(이름·전화 안 받음=프라이버시).

### B2 — Here's your zone (핵심)
**목적:** 어느 **구역**에 있는지 안내 + 그 구역서 송장 호수로 빨리 찾게. (콕 아님, 구역)

**요소 / copy:**
- 큰 강조: **"Zone B"** (색 강조)
- 부제: **"2 packages for 102-1204"**
- **구역 지도** (mailroom 평면 도식)에서 Zone B 하이라이트
- 안내: **"Look for unit 1204 on the label in Zone B."**
- 버튼 2개 (같은 크기, 명확한 분기):
  - **"Print location"** (종이로 위치 출력 — 들고 가고 싶은 사람)
  - **"Done"** (그냥 보고 감)
- **둘 중 무엇을 눌러도 → 수령 처리(등록 해제) + ENTRY 복귀**

**행동:** 위치 확인 → Print 또는 Done → 수령 완료 + 리셋.
**근거:** 위치 인덱싱(구역까지), 색+이름 병기(저시력/색각), 송장 호수로 최종 식별(생김새 불요), Hick(명확한 2분기), "워크플로우 완료=수령 인식"(센서 없이 상태 갱신), privacy 리셋.

---

## 6. 인터랙션 / 상태 노트

- **상태 모델(데모용, 메모리 변수):**
  - `mode`: `entry | store | find`
  - STORE: 선택된 `zone`, 스캔 누적 `scannedList[]`(호수 문자열)
  - FIND: 입력된 `building`, `unit` → 가짜 데이터로 `zone` 매핑(예: 102→Zone B)
- **데모 단순화:** FIND에서 동·호수 입력 시, 미리 박아둔 매핑으로 즉시 Zone 안내(실제 DB 불필요).
- **수령 처리:** B2에서 Print/Done 누르면 해당 호수 등록 해제(데모에선 콘솔/상태만 갱신) → ENTRY.
- **자동 리셋:** A3, B2 완료 후 ENTRY로. (유휴 타임아웃도 가능)
- **브라우저 storage 금지** (sandbox 호환). 전부 메모리 변수.

---

## 7. 데모 해피패스 (발표 ④, 약 1:50)

1. ENTRY → 🔵 **Find my package**
2. B1: `102` → `1204` 입력 → 확인
3. B2: **"Zone B — 2 packages for 102-1204"** + 지도 하이라이트
4. **"Done"** → 수령 완료 → ENTRY
5. (여유 되면) 🟠 **Store** 짧게: Zone B 선택 → "Scan" 두세 번(띡띡) → "Done"

각 화면 클릭하며 한 단어씩: 큰 키(Fitts) / 한 화면 한 결정(Hick) / 색+이름(접근성) / 정면 스캔(OCR 신뢰) / 리셋(privacy).

---

## 8. 제작 메모 (기술)

- HTML/CSS/JS 단일 파일 또는 컴포넌트로. 라이트 테마, 세로 비율 고정 프레임.
- 키패드·스캔·화면전환은 vanilla JS state machine으로 충분.
- 스캐너 뷰파인더·구역 지도·체크 아이콘은 placeholder(도형/박스)로 두고 나중에 디자인.
- 디자인 시스템(getdesign 등)으로 색·폰트·컴포넌트 입히되, **이 문서의 화면 구성·copy·플로우는 유지**.
- 접근성: 고대비, 큰 타겟, 텍스트 충분히 크게.

---

## 9. 발표와의 연결 (참고)

- ENTRY의 두 모드 = "하나의 키오스크가 보관+찾기 둘 다" (통합성)
- A2 연속 스캔 = "계산대처럼, 가두지 않고 등록만"
- B2의 Zone(색+이름) = "라커 없이 구역화만으로"
- 구역 크기 조절(페인트) = "라커는 하드웨어 고정, 우리는 바닥선" (What's different)

---

## 10. Design system mapping (HP via getdesign)

> `npx getdesign@latest add hp` 로 받은 `DESIGN.md`를 기반으로 매핑.
> HP의 시그니처(흰 캔버스 + Electric Blue + 청남 슬랩 + 청색 셰브론) 위에 키오스크의 두 모드(반대 색) 규칙을 얹는다.

### 10.1 모드 ↔ 브랜드 색 매핑

| 모드 | 역할 | HP 토큰 | Hex |
|---|---|---|---|
| **Find (거주자)** | 메인 CTA · 1차 액션 | `colors.primary` (HP Electric Blue) | `#024AD8` |
| **Store (택배기사)** | 반대 색 · 2차 액션 | `colors.bloom-coral` | `#FF5050` |
| Find pressed | | `colors.primary-deep` | `#0E3191` |
| Store pressed | | `colors.bloom-deep` | `#B3262B` |

근거: HP는 원래 단일 시그널 색(Electric Blue)을 쓰지만, 키오스크 스펙(§1)은 "Store/Find 반대 색"을 요구. HP 팔레트 안에서 가장 대비 강한 따뜻한 톤이 `bloom-coral`이라 채택. 두 색은 화면당 동시에 등장하지 않게(모드가 분리되어 있어 자연스럽게 충돌 없음).

### 10.2 Surface · Text

- **Canvas**: `#FFFFFF` (모든 화면 기본 배경)
- **Cloud**: `#F7F7F7` (ENTRY hero, 키패드 보조 면, 지도 배경)
- **Ink**: `#1A1A1A` (본문/헤드라인, 유틸리티 스트립, A3 슬랩 배경)
- **Graphite / Charcoal**: `#636363` / `#3D3D3D` (보조 텍스트, 메타)
- **Hairline / Steel**: `#E8E8E8` / `#C2C2C2` (테두리)
- 다크 슬랩 위 텍스트는 `#FFFFFF`, 부가 카피는 `rgba(255,255,255,0.55)`로.

### 10.3 Typography

- 폰트: **Forma DJR Micro** (상용) → 오픈 대체로 **Inter** 사용. 본문 라인하이트 1.38, 디스플레이 1.0.
- 위계: Hero 64/700, Section headline 44/500, Card 28/500, Body 16/400, Caption 14/400.
- 버튼 라벨은 **UPPERCASE + 0.7~0.8px letter-spacing** (HP만의 특징).

### 10.4 Shape · Elevation

- **버튼/입력**: `radius 4px` (HP 시그니처: 인터랙티브는 날카롭게)
- **카드/사진 프레임**: `radius 16px` (카드는 부드럽게)
- **배지/칩**: `radius 8px`
- 그림자는 *Soft Lift* (`0 2px 8px rgba(26,26,26,0.08)`) 한 단계만. 카드/모달 외에는 평면.

### 10.5 Signature: chevron

- ENTRY hero 좌우에 청색 평행 슬래시 두 개(`#024AD8`, 0 radius, ~22° skew). HP 워드마크의 시그니처를 키오스크 어휘로 차용.
- 셰브론은 **hero에만** 등장. 카드/버튼 안에 들어가면 노이즈가 되므로 금지.

### 10.6 Component 매핑 (this kiosk ↔ HP DESIGN.md)

| 키오스크 요소 | HP component |
|---|---|
| ENTRY 모드 버튼 | `card-product` (24px padding, 16px radius, Soft Lift) |
| Zone 카드 (A1) | `card-product` 변형 (스트라이프 + name + count) |
| 다크 슬랩 (A3) | `promo-strip-dark` |
| 유틸리티 스트립 | `utility-strip` |
| Back, Add manually | `button-outline-ink` |
| Done(Store), Save(modal) | `button-store` (= `button-primary` 변형, bloom-coral) |
| OK(B1), Print/Done(B2) | `button-primary` (Electric Blue) |
| 모달 | `card-product` + Floating Modal shadow (`0 8px 24px`) |

---

## 11. Interaction additions (v2)

원본 §3~§7 플로우는 그대로. 아래는 데모/현장 테스트 중에 추가된 인터랙션.

### 11.1 A2 — 스캔 결과는 "동-호수" 형식

- 스캔 직후 목록 칩에 **`102-1204`** 형태(building-unit)로 표시. 단지 내 다른 동 택배도 같은 mailroom으로 올 수 있으므로 동 정보가 필수.
- 카운터는 그대로 `"7 scanned"`.
- 목록은 **newest-first**로 정렬(가장 최근 스캔이 맨 위) — 기사가 방금 한 스캔을 즉시 확인.

### 11.2 A2 — "Add manually" (원본 "re-scan 안내" 대체/보강)

- 원본 §4 A2의 "Couldn't read — re-scan"은 OCR 실패 시 **자동으로** 뜨는 메시지였음. 사용자가 누를 버튼이 아님.
- 현장에서 라벨이 찢어졌거나 손글씨인 경우 대응이 필요해 **`Add manually` 보조 버튼**을 Done 옆에 추가.
- 탭 → 모달 (Building 3자리 → Unit 3~4자리 키패드) → Save → 목록에 추가.
- 모달 라벨/Save 키는 Store 모드 색(bloom-coral).

### 11.3 A2 — 스캔 항목 탭하여 수정/삭제

- 스캔된 칩(예: `102-1204`)을 탭하면 같은 모달이 **Edit entry**로 열림.
- 기존 값 프리필 + `Save`로 덮어쓰기 + `Delete entry` 링크로 삭제.
- 백드롭 클릭 또는 ✕ 으로 취소.
- 근거: 연속 스캔은 빠르지만 한 건 잘못 들어가면 전체를 다시 하기 부담. 인라인 편집으로 복구 비용 최소화.

### 11.4 B1 — 호수 3자리부터 OK 활성

- 원본은 호수 4자리 가정. 실제 단지에는 **3자리 호수(예: 204호)**도 흔함.
- 동 3자리 + **호수 3자리부터** OK 버튼 활성. 4자리까지 입력도 허용.
- 매칭 시 3자리 입력이면 자동으로 `0` padding 한 4자리로도 매칭 시도(예: `204` → `0204`).

### 11.5 Data model (메모리 변수, v2)

```js
state = {
  mode: "entry" | "store" | "find",
  zone: "A" | "B" | "C" | "D" | null,
  scannedList: ["102-1204", "102-0907", ...],  // ← 형식 변경
  building: "102",
  unit: "1204" | "204",   // ← 3 or 4 digits
  activeField: "building" | "unit",
}

modalState = {
  open: boolean,
  mode: "add" | "edit",
  index: number,    // edit 모드일 때 scannedList의 인덱스
  building, unit, activeField,
}
```

### 11.6 Demo happy path (v2)

1. ENTRY → Find → `102` → `204` (3자리) → OK 활성 → Zone 결과
2. (보조) Store → Zone B → Scan 3회(`102-1204`, `103-0907`, `102-1511`) → 두 번째 항목 탭 → 호수 수정 → Save → Done
