---
name: screen-analysis
description: Frontend URL을 기준으로 실제 화면 기능을 분석하고 Action 목록 문서를 생성한다.
argument-hint: "<Frontend URL>"
user-invocable: true
disable-model-invocation: true
---

# 화면 기능 분석 Skill

## 1. 역할

입력된 Frontend URL을 기준으로
실제 화면에서 실행 가능한 기능을 찾고
SCREEN 문서를 생성한다.

이 Skill은 화면 기능 분석만 수행한다.

Frontend 기능 상세 분석,
API 규격 분석,
Backend 기능 분석은 수행하지 않는다.


## 2. 입력

사용자가 Frontend URL을 입력한다.

예:

`/equipment/search`

또는

`http://localhost:3000/equipment/search`


## 3. 적용 규칙

반드시 다음 Rule을 따른다.

- `01-screen-analysis-scope.md`
- `02-screen-action-discovery.md`
- `03-screen-popup-navigation.md`

Rule과 Skill의 내용이 충돌하는 경우
Rule을 우선한다.


## 4. 전체 실행 흐름

화면 기능 분석은 다음 순서로 수행한다.

Frontend URL 확인

→ Chrome DevTools MCP로 화면 접근

→ 실제 화면 확인

→ Action 후보 탐색

→ Event 확인

→ 필요한 경우 Handler 확인

→ 필요한 경우 Code Index MCP로 Source Path 확인

→ Popup / Navigation 확인

→ Action ID 부여

→ SCREEN 문서 생성

→ 종료


## 5. STEP 1 — URL 확인

사용자가 입력한 Frontend URL을
현재 분석 대상으로 설정한다.

현재 URL과 직접 관련되지 않은
다른 화면을 임의로 분석하지 않는다.


## 6. STEP 2 — 화면 접근

Chrome DevTools MCP를 사용하여
입력된 URL의 실제 화면을 확인한다.

확인 대상:

- 현재 URL
- 화면 제목
- 주요 화면 영역
- 실제 표시되는 UI
- 화면 진입 시 발생하는 동작

화면 Runtime을
Action 탐색의 기본 근거로 사용한다.


## 7. STEP 3 — Action 후보 탐색

현재 화면에서 실행 가능한 기능을 찾는다.

예:

- Button
- Link
- Icon
- Menu
- Tab
- Grid
- Checkbox
- Radio
- Select
- Form Submit
- Pagination
- Popup
- Navigation
- 화면 초기 실행 기능

단순 DOM Element를
모두 Action으로 등록하지 않는다.

Action 판정은
`02-screen-action-discovery.md`를 따른다.


## 8. STEP 4 — Runtime 동작 확인

안전하게 확인 가능한 Action은
Runtime에서 동작을 확인할 수 있다.

확인 가능한 정보:

- Event
- 화면 변화
- Modal / Layer Popup
- Navigation
- Window Popup
- Network 발생 여부

단, 현재 SCREEN 단계에서는
Network의 API 규격을 분석하지 않는다.


## 9. 상태 변경 Action 보호

다음과 같이 데이터 또는 업무 상태를
변경할 가능성이 있는 Action은
사용자의 명시적인 허용 없이 실행하지 않는다.

예:

- 저장
- 등록
- 수정
- 삭제
- 승인
- 반려
- 확정
- 업무 취소
- 전송

이러한 Action은
화면에 존재하는 기능으로 식별하는 것까지만 수행한다.


## 10. STEP 5 — Handler 확인

Action과 연결된 Handler를
확인할 수 있는 경우 기록한다.

예:

검색 버튼

→ Click

→ `handleSearch`

Handler 이름을 확인하기 위해
관련 Frontend Source를 탐색할 수 있다.

그러나 Handler 내부의 상세 로직은
분석하지 않는다.


## 11. STEP 6 — Runtime 화면과 Local Source 연결

Chrome DevTools MCP에서 확인한 Runtime 화면을
현재 프로젝트의 Local Frontend Source와 연결한다.

개발 서버에서 전달되는 Build 결과물이나
Minified JavaScript를 주 분석 대상으로 사용하지 않는다.

Chrome DevTools MCP는 Runtime 확인에 사용하고,
Code Index MCP는 Local Original Source 탐색에 사용한다.


### 11.1 Source 탐색 기준

다음 정보를 단서로 Local Source를 탐색한다.

우선순위:

1. 현재 Frontend URL / Route
2. 화면 또는 Page Component
3. 화면에 표시되는 고유 Text
4. Component 이름
5. Event / Handler 이름
6. 관련 Symbol / Reference

가능하면 하나의 단서만으로 Source를 확정하지 않고
둘 이상의 근거를 확인한다.


### 11.2 URL / Route 기반 탐색

현재 Runtime URL을 먼저 확인한다.

예:

`/equipment/search`

Local Source에서 해당 URL과 연결된
Route 또는 Page Component를 찾는다.

예:

`/equipment/search`
→ Router 설정
→ `EquipmentSearch`
→ `src/views/equipment/EquipmentSearch.vue`

Route를 확인할 수 있는 경우
이를 화면 Source 탐색의 우선 근거로 사용한다.


### 11.3 화면 Component 확인

Route에서 Page Component를 찾은 경우
해당 Component를 현재 화면 Source의 시작점으로 사용한다.

필요한 경우 현재 화면과 직접 연결된
하위 Component까지만 탐색할 수 있다.

예:

`EquipmentSearch.vue`

→ `SearchCondition.vue`

→ `EquipmentGrid.vue`

현재 화면과 관련 없는 Component까지
탐색 범위를 확장하지 않는다.


### 11.4 Action과 Handler 연결

Chrome DevTools MCP에서 발견한 Action을
Local Source의 UI Element / Event / Handler와 연결한다.

예:

Runtime:

검색 버튼

Local Source:

`<button @click="handleSearch">검색</button>`

연결 결과:

검색 버튼
→ Click
→ `handleSearch`
→ `EquipmentSearch.vue`

이 단계에서는 `handleSearch` 내부 로직을
상세 분석하지 않는다.


### 11.5 Code Index MCP 사용

Code Index MCP는 다음 목적으로 사용한다.

- Route 관련 Source 탐색
- Page Component 탐색
- Handler Symbol 탐색
- Component Reference 확인
- Source Path 확인

Code Index 결과는 Source 후보를 찾기 위한 근거이다.

검색 결과에 동일하거나 유사한 Handler가 여러 개 존재하면
이름만 보고 현재 화면의 Handler라고 확정하지 않는다.


### 11.6 Source 일치 검증

Source 후보를 발견하면
현재 Runtime 화면과 실제 Local Source가
일치하는지 확인한다.

가능한 검증 정보:

- URL / Route
- 화면명
- 화면 Text
- Component 구조
- UI Element
- Event
- Handler
- Import / Component 관계

근거가 충분한 경우에만
Handler와 Source Path를 확정한다.


### 11.7 Runtime과 Local Source 불일치

개발 서버와 Local Source의 버전 또는 Branch가
다를 가능성을 고려한다.

Runtime 화면에는 존재하지만
Local Source에서 대응되는 코드를 확인할 수 없는 경우
억지로 연결하지 않는다.

다음과 같이 기록한다.

- Handler: `확인되지 않음`
- Source Path: `확인되지 않음`

필요한 경우 미확인 사유에 다음과 같이 기록한다.

`Runtime 화면과 Local Source의 일치 여부 확인 필요`


### 11.8 분석 경계

Source Path를 확인한 뒤
현재 SCREEN 분석에서는 더 깊게 추적하지 않는다.

허용:

Runtime Action
→ Route
→ Component
→ Event
→ Handler
→ Source Path

금지:

Handler
→ Validation
→ Parameter 생성
→ State 처리
→ API 호출
→ Frontend Business Logic

위 상세 분석은
Frontend 기능 분석 단계에서 수행한다.


## 12. Code Index 사용 제한

SCREEN 분석에서 Code Index MCP는
필요한 경우에만 사용한다.

전체 Repository를 탐색하지 않는다.

현재 화면과 직접 관련된
Component / Handler / Source Path 확인에만 사용한다.

다음 분석으로 확장하지 않는다.

- Handler 내부 Business Logic
- API 구현
- Controller
- Service
- Mapper
- SQL


## 13. STEP 7 — Popup / Navigation 확인

Popup 및 화면 이동은
`03-screen-popup-navigation.md`를 따른다.

### Modal / Layer Popup

현재 화면의 일부로 취급한다.

Modal 내부의 Action도
현재 SCREEN 분석에 포함할 수 있다.


### Window Popup / 새 Tab

다음 정보까지만 확인한다.

- Action
- Event
- Handler
- Source Path
- Target URL
- 확인 가능한 Parameter

새 화면 내부는 분석하지 않는다.


### Navigation

다른 화면으로 이동하는 경우

- Action
- Event
- Handler
- Source Path
- Target URL
- 확인 가능한 Parameter

까지만 기록한다.

이동한 화면은 자동으로 분석하지 않는다.


## 14. STEP 8 — Action ID 부여

확인된 화면 기능에
Action ID를 부여한다.

형식:

`ACT-001`

`ACT-002`

`ACT-003`

동일 업무 기능에 여러 Trigger가 존재하는 경우
하나의 Action으로 표현할 수 있다.

예:

ACT-002 검색

Trigger:

- 검색 Button Click
- 검색조건 Enter


## 15. STEP 9 — 근거 확인

각 Action은 가능한 범위에서
확인된 근거를 기반으로 작성한다.

우선적으로 사용:

1. Chrome DevTools Runtime
2. Frontend Source Code
3. Code Index 탐색 결과

확인되지 않은 정보는
추측하지 않는다.

확인할 수 없는 경우:

`확인되지 않음`

으로 기록한다.


## 16. STEP 10 — SCREEN 문서 생성

분석 결과를 화면별 디렉터리에
SCREEN 문서로 생성한다.

화면명은 가능한 경우
Frontend URL 또는 Route를 기준으로 결정한다.

예:

Frontend URL:

`/equipment/search`

화면명:

`equipment-search`

저장 위치:

`docs/analysis/equipment-search/`

SCREEN 문서:

`docs/analysis/equipment-search/SCREEN-equipment-search.md`

필요한 디렉터리가 존재하지 않는 경우 생성한다.

현재 단계에서는
`frontend`, `api`, `backend` 분석 문서를 생성하지 않는다.

기존 동일 이름의 SCREEN 문서가 존재하는 경우
임의로 덮어쓰지 않는다.


## 17. SCREEN 문서 기본 구조

문서는 다음 구조를 사용한다.

### 1. 화면 정보

- 화면명
- Frontend URL
- 분석 기준
- 분석 일시


### 2. 화면 기능 요약

현재 화면에서 확인된
주요 기능을 요약한다.


### 3. Action 목록

| Action ID | 기능명 | UI 유형 | Event | Handler | Source Path | 상태 |
|---|---|---|---|---|---|---|

상태 기본값:

`미분석`


### 4. Action 상세

각 Action별로 다음 정보를 기록한다.

- Action ID
- 기능명
- Trigger
- UI 위치
- Event
- Handler
- Source Path
- Popup 여부
- Navigation 여부
- Target URL
- 확인 가능한 Parameter
- 근거
- 미확인 항목


### 5. Popup / Navigation

확인된 Popup 및
화면 이동 정보를 기록한다.


### 6. 미확인 항목

Runtime 또는 Source에서
확인할 수 없었던 정보를 기록한다.


### 7. 분석 경계

이번 분석에서 수행하지 않은 영역을 명시한다.

- Frontend Business Logic
- API 규격
- Backend
- Database


### 8. 다음 분석 후보

Frontend 상세 분석이 가능한
Action ID 목록을 표시한다.

단, 다음 분석을 자동으로 시작하지 않는다.


## 18. 금지 사항

SCREEN 분석에서는 다음을 수행하지 않는다.

- Frontend Business Logic 상세 분석
- Validation 상세 분석
- Parameter 생성 과정 상세 분석
- API 규격 분석
- Backend 상세 분석
- Oracle 분석
- SQL 분석
- 전체 Repository 탐색
- 다른 화면 자동 분석
- 다음 분석 단계 자동 실행


## 19. 종료 조건

SCREEN 문서가 생성되면
즉시 화면 기능 분석을 종료한다.

사용자가 Action ID를 선택하기 전까지
Frontend 기능 분석을 시작하지 않는다.


## 20. 완료 보고

SCREEN 분석 완료 후 다음 정보만 간단히 보고한다.

- 생성된 SCREEN 문서
- 발견한 Action 수
- Action ID 목록
- 미확인 항목 존재 여부

그리고 종료한다.