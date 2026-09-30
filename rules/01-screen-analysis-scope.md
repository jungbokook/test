---
name: screen-analysis-scope
description: 화면 기능 분석 단계에서 허용되는 분석 범위와 종료 기준을 정의한다.
---

# 화면 기능 분석 범위 규칙

## 1. 목적

화면 기능 분석은 사용자가 입력한 Frontend URL을 기준으로
실제 화면에서 실행 가능한 기능을 식별하는 단계이다.

이 단계의 목적은 화면 기능 목록을 만드는 것이다.

Frontend 비즈니스 로직이나
Backend 기능을 상세 분석하는 단계가 아니다.


## 2. 입력

화면 기능 분석의 입력은 Frontend URL이다.

예:

`/equipment/search`


## 3. 분석 대상

현재 화면에서 다음과 같은 기능을 확인할 수 있다.

- Button
- Link
- Icon
- Menu
- Tab
- Grid Row / Cell
- Checkbox
- Radio
- Select
- Input
- Form Submit
- Pagination
- Popup
- Modal
- 화면 진입 시 자동 실행되는 기능

단순히 화면에 존재하는 모든 요소를
기능으로 등록하지 않는다.

실제 동작과 연결된 화면 요소를 대상으로 한다.


## 4. 허용되는 분석 범위

화면 기능을 식별하기 위해
다음 정보까지 확인할 수 있다.

- 화면 요소
- 화면 표시명
- UI Type
- Event
- Handler
- Frontend Source Path
- Popup 여부
- Navigation 여부

예:

검색 버튼
→ Click
→ handleSearch
→ `src/.../EquipmentSearch.tsx`

여기까지는 화면 기능 분석 범위에 포함한다.


## 5. Handler 분석 경계

Handler를 발견하더라도
Handler 내부의 상세 로직으로 자동 확장하지 않는다.

허용:

검색 버튼
→ onClick
→ handleSearch
→ Source Path 확인

현재 단계에서 분석하지 않음:

handleSearch
→ Validation
→ Parameter 생성
→ State 변경
→ API 호출
→ Backend

Handler의 존재와 위치를 확인하는 것과
Handler 내부 비즈니스 로직을 분석하는 것을 구분한다.


## 6. Backend 호출 발견

화면 기능을 확인하는 과정에서
API 또는 Network 호출이 보이더라도
API 규격이나 Backend를 상세 분석하지 않는다.

발견 사실이 현재 화면 기능 식별에 필요한 경우
간단한 참고 정보로 남길 수 있지만
추적 분석으로 확장하지 않는다.


## 7. 금지되는 분석

화면 기능 분석에서는 다음을 수행하지 않는다.

- Frontend Validation 상세 분석
- Parameter 생성 과정 분석
- Frontend Business Logic 상세 분석
- State 처리 상세 분석
- API 규격 분석
- Request / Response 규격 분석
- Controller 분석
- Service 분석
- Backend Business Logic 분석
- Mapper 분석
- MyBatis Mapper XML 분석
- SQL 분석
- Oracle 분석
- SAP / External System 상세 분석


## 8. Action ID

발견한 화면 기능에는
고유한 Action ID를 부여한다.

형식:

`ACT-001`

`ACT-002`

`ACT-003`

하나의 화면 기능 목록 안에서
동일한 Action ID를 중복 사용하지 않는다.


## 9. 미확인 정보

확인되지 않은 Handler나 Source Path를
추측해서 작성하지 않는다.

확인할 수 없는 경우:

`확인되지 않음`

으로 기록한다.


## 10. 출력

분석 결과는 화면 기능 목록 문서로 작성한다.

문서에는 이후 Frontend 기능 분석에서
사용자가 분석 대상을 선택할 수 있도록
Action ID를 명확하게 표시한다.


## 11. 종료 조건

화면 기능 목록 문서 생성이 완료되면
화면 기능 분석을 종료한다.

다음 단계인 Frontend 기능 분석을
자동으로 시작하지 않는다.

다음 분석 대상은 사용자가
Action ID를 직접 선택한다.


## 12. 검증

Rule 구축 단계의 테스트에서는
마지막에 다음 문구를 출력한다.

`RULE_CHECK: SCREEN_ANALYSIS_SCOPE_APPLIED`