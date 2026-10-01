claude mcp remove code-index


claude mcp add --scope project code-index -- uvx --native-tls code-index-mcp --project-path "C:\work\GIPMS"



## 8. 문서 저장 위치

기존 내용...


## 9. Source Root 구분

현재 분석 Root 아래에 여러 Frontend / Backend 프로젝트가 존재한다.

### Backend

Backend 프로젝트 패턴:

`gipms-api-*`

- `gipms-api-*`에 해당하는 디렉터리는 Backend로 취급한다.
- Backend 프로젝트가 여러 개 존재할 수 있다.

### Frontend

Frontend 프로젝트 패턴:

`gipms-*`

단, `gipms-api-*`는 Frontend에서 제외한다.

따라서 프로젝트 구분 우선순위는 다음과 같다.

1. `gipms-api-*` → Backend
2. `gipms-*` 중 `gipms-api-*`가 아닌 디렉터리 → Frontend

### 분석 시 탐색 범위

- SCREEN 분석 → Frontend 프로젝트 우선
- FE 분석 → Frontend 프로젝트 우선
- API 분석
  - FE 호출 Source → Frontend
  - Controller / Request DTO / Response DTO → Backend
- BE 분석 → Backend 프로젝트 우선

여러 Backend 프로젝트가 존재하는 경우
모든 프로젝트를 깊게 분석하지 않는다.

먼저 `gipms-api-*` 범위에서 대상 API의
Controller Mapping 또는 관련 Source를 탐색하고,
실제 관련성이 확인된 Backend 프로젝트만 추적한다.

프로젝트 이름만으로 실제 호출 대상을 확정하지 않는다.
실제 Source와 호출 관계를 Evidence로 사용한다.


## 10. 기존 다음 항목

...