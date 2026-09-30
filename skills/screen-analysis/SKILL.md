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


## 11. STEP 6 — Source Path 확인

Handler 또는 Component의 Source Path가 필요한 경우
Code Index MCP를 사용할 수 있다.

Code Index MCP의 목적은
관련 Source 위치를 찾고 범위를 좁히는 것이다.

확인 대상:

- Symbol
- Handler
- Component
- Source Path
- Reference

Code Index 결과만으로
Business Logic을 해석하지 않는다.


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

분석 결과를
SCREEN 문서로 생성한다.

권장 파일명:

`SCREEN-{화면명}.md`

예:

`SCREEN-equipment-search.md`


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