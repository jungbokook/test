---
name: be-test-controller
description: Backend URL에서 Controller와 Service를 찾은 뒤 ServiceImpl 전체 파일을 읽지 않고 대상 Method 범위만 읽어 Mapper 호출명까지 확인하는 성능 테스트 Skill.
argument-hint: "<HTTP Method|UNKNOWN> <Backend URL>"
allowed-tools: Read, Grep
---

# BE Fast Search Test - Partial Method Read

## 0. 목적

이 Skill은 최종 BE 분석을 수행하지 않는다.

이번 테스트의 실행 범위:

Controller
→ Service
→ ServiceImpl 위치
→ ServiceImpl 대상 Method 위치
→ 대상 Method 범위만 Read
→ Mapper 호출명 확인
→ STOP

핵심 테스트:

ServiceImpl 전체 파일을 읽지 않고
실제 호출되는 Method 부분만 읽었을 때
검색 시간이 얼마나 줄어드는지 측정한다.

---

# 1. 입력

사용법:

/be-test-controller <HTTP Method|UNKNOWN> <Backend URL>

예:

/be-test-controller POST /material/create

/be-test-controller UNKNOWN /material/create

입력:

- HTTP Method
- Backend URL

---

# 2. 검색 범위

Backend 프로젝트만 검색한다.

대상:

gipms-api-*

제외:

- docs/**
- sample/**
- **/sample/**
- target/**
- build/**
- *.jar
- test/**
- **/test/**
- generated/**
- node_modules/**

Frontend 프로젝트는 검색하지 않는다.

---

# 3. 허용 도구

사용:

- Grep
- Read

사용 금지:

- Glob
- Code Index
- Agent
- BE-REFERENCE

---

# 4. 최우선 성능 규칙

이번 테스트에서는 속도가 최우선이다.

반드시 다음 규칙을 따른다.

1. 동일 검색을 반복하지 않는다.
2. 이미 찾은 파일을 다시 찾지 않는다.
3. 현재 Backend 프로젝트부터 검색한다.
4. 찾지 못한 경우에만 다른 gipms-api-*로 확대한다.
5. Controller가 확정되면 URL 검색을 중단한다.
6. Service가 확정되면 Service 검색을 중단한다.
7. ServiceImpl이 확정되면 구현체 검색을 중단한다.
8. ServiceImpl 전체 파일 Read를 금지한다.
9. ServiceImpl 대상 Method 위치를 먼저 찾는다.
10. 대상 Method 주변의 필요한 범위만 Read한다.
11. Mapper 호출명이 확인되면 즉시 STOP한다.
12. Mapper Interface는 절대 찾지 않는다.
13. Evidence를 위해 두 번째 검색을 하지 않는다.

내부적으로 기억:

KNOWN_FILES
KNOWN_SYMBOLS
VISITED_FILES
VISITED_METHODS

---

# STEP 1. URL 분리

Backend URL을 path 단위로 나눈다.

예:

/material/create

상위 path:

/material

마지막 segment:

/create

Controller 검색에서는
가능하면 식별력이 높은 상위 path를 먼저 사용한다.

---

# STEP 2. Controller 찾기

검색 범위:

gipms-api-*/**/*Controller.java

URL의 상위 path를 Grep한다.

예:

/material

후보 Controller가 발견되면
Backend 전체 URL 검색을 중단한다.

후보 Controller 파일만 확인한다.

상위 path로 확정할 수 없을 때만
마지막 segment를 추가 검색한다.

---

# STEP 3. Controller Mapping 검증

후보 Controller에서:

class-level mapping

+

method-level mapping

을 조합한다.

예:

@RequestMapping("/material")

+

@PostMapping("/create")

=

POST /material/create

입력 URL과 정확히 일치하는 Controller Method를 확정한다.

---

# STEP 4. HTTP Method

입력이:

GET
POST
PUT
DELETE
PATCH

이면 annotation과 일치해야 한다.

UNKNOWN이면 Controller annotation에서 확정한다.

동일 URL에 여러 Method가 존재하면
임의 선택하지 않는다.

AMBIGUOUS로 종료한다.

---

# STEP 5. Controller Method 확인

확인:

- Controller Class
- Controller Method
- HTTP Method
- Evidence

Controller Method가 확정되면
다른 Controller는 탐색하지 않는다.

---

# STEP 6. Service 호출 확인

확정된 Controller Method 내부에서
실제로 호출되는 Service만 확인한다.

예:

materialService.create(request);

기록:

Service Variable:

materialService

Service Method:

create

---

# STEP 7. Service Type 확인

Controller의 이미 확인된 내용에서
Service Variable Type을 찾는다.

예:

private final MaterialService materialService;

결과:

Service Type:

MaterialService

이 정보를 찾기 위해
Controller를 다시 Grep하지 않는다.

---

# STEP 8. Service 찾기

Service 경로를 모르는 경우
현재 Backend 프로젝트에서 정확한 Type만 Grep한다.

예:

interface MaterialService

현재 프로젝트에서 발견되면 STOP SEARCH.

다른 gipms-api-*는 검색하지 않는다.

현재 프로젝트에 없을 때만 범위를 확대한다.

---

# STEP 9. Service Method 확인

Controller가 호출한 Method 선언만 확인한다.

예:

create(...)

확인:

- Service
- Service Method
- Parameter
- Return Type
- Evidence

다른 Service Method는 분석하지 않는다.

---

# STEP 10. ServiceImpl 찾기

Service가 interface인 경우:

implements MaterialService

를 검색한다.

먼저 현재 Backend 프로젝트만 검색한다.

예:

MaterialServiceImpl implements MaterialService

구현체가 발견되면
구현체 검색을 즉시 중단한다.

중요:

이 단계에서 ServiceImpl 파일 전체를 Read하지 않는다.

파일 경로만 확보한다.

---

# STEP 11. ServiceImpl 대상 Method 위치 찾기

ServiceImpl 경로가 확정되면
해당 파일 하나에서만 Controller가 호출한
정확한 Method 이름을 Grep한다.

예:

Controller 호출:

materialService.create(...)

그러면 ServiceImpl 파일에서:

create(

를 찾는다.

가능하면 선언 형태를 우선 확인한다.

예:

public Result create(

또는:

public void create(

또는:

create(

목표는 Method 시작 위치를 찾는 것이다.

ServiceImpl 전체 파일을 Read하지 않는다.

다른 파일도 검색하지 않는다.

---

# STEP 12. Method 후보 검증

동일 이름 Method가 여러 개 존재할 수 있다.

예:

create(Request request)

create(Request request, User user)

이 경우 Service Interface의 Method signature와 비교해서
실제 구현 Method를 확정한다.

확정된 Method의 시작 line을 기록한다.

다른 overload는 분석하지 않는다.

---

# STEP 13. 부분 Read

확정된 ServiceImpl Method의 시작 위치부터
필요한 범위만 Read한다.

절대 ServiceImpl 전체 파일을 읽지 않는다.

초기 Read 범위는:

Method 시작 line부터 약 80줄 이내

를 우선 사용한다.

예:

Method 시작:

line 420

초기 확인:

420 ~ 500

Method가 80줄 이내에서 종료되면
추가 Read를 하지 않는다.

---

# STEP 14. Method가 80줄을 초과하는 경우

초기 범위에서 Method 종료가 확인되지 않은 경우에만
다음 범위를 추가 Read한다.

예:

420 ~ 500
↓
501 ~ 580

한 번에 전체 파일을 읽지 않는다.

필요한 만큼만 순차적으로 확장한다.

Method 종료가 확인되는 즉시
추가 Read를 중단한다.

---

# STEP 15. Mapper 호출명 추출

부분 Read한 Method 범위에서
직접 호출되는 Mapper 호출명만 확인한다.

예:

materialMapper.selectMaterial(param);

materialMapper.insertMaterial(param);

기록:

- materialMapper.selectMaterial()
- materialMapper.insertMaterial()

Mapper Type은 찾지 않는다.

Mapper Interface도 찾지 않는다.

---

# STEP 16. 여러 Mapper 호출

Method 내부에 여러 Mapper 호출이 직접 존재하면
모두 기록한다.

예:

materialMapper.selectMaterial(...);

historyMapper.insertHistory(...);

결과:

- materialMapper.selectMaterial()
- historyMapper.insertHistory()

추가 Mapper 파일 검색은 하지 않는다.

---

# STEP 17. 조건 분기

부분 Read 범위에서 조건문이 직접 보이고
그 안에서 Mapper가 호출되면 호출 구조만 유지한다.

예:

if (exists) {

    materialMapper.updateMaterial(...);

} else {

    materialMapper.insertMaterial(...);

}

결과:

IF exists
├─ materialMapper.updateMaterial()
└─ materialMapper.insertMaterial()

조건값의 출처는 추적하지 않는다.

---

# STEP 18. Local Method

현재 테스트에서는 Local/private Method 내부로 들어가지 않는다.

예:

create()
→ validate()
→ save()

validate()와 save() 내부는 분석하지 않는다.

현재 대상 Method에 직접 작성된 Mapper 호출만 확인한다.

Local Method 위치도 검색하지 않는다.

---

# STEP 19. 다른 Service 호출

대상 Method에서 다른 Service가 호출되더라도
현재 테스트에서는 내부로 들어가지 않는다.

예:

commonService.validate(...)

호출이 있다는 사실만 볼 수 있지만
CommonService 파일을 찾지 않는다.

---

# STEP 20. Evidence

Evidence는 다음까지만 출력한다.

- Controller
- Service
- ServiceImpl 대상 Method

형식:

Project Root 기준 상대경로:라인범위

절대경로는 사용하지 않는다.

예:

gipms-api-material/src/main/java/.../MaterialController.java:42-58

gipms-api-material/src/main/java/.../MaterialService.java:15-18

gipms-api-material/src/main/java/.../MaterialServiceImpl.java:420-468

Evidence 확보를 위해 추가 검색하지 않는다.

---

# STEP 21. 강제 STOP

ServiceImpl 대상 Method의 직접 Mapper 호출명을 확인하면
즉시 종료한다.

이후 Grep/Read를 수행하지 않는다.

절대 수행하지 않는다.

- Mapper Type 검색
- Mapper Interface 검색
- Mapper Java 파일 검색
- Mapper Method 선언 검색
- MyBatis XML 검색
- namespace 검색
- statement id 검색
- SQL 검색
- Local Method 추적
- 다른 Service 내부 추적
- DB 분석
- Oracle Metadata
- SAP/RFC 검색
- 외부 API 검색
- BE-REFERENCE 읽기
- Markdown 문서 생성

---

# STEP 22. 출력

아래 형식으로만 출력한다.

## BE Fast Search Result

- URL:
- HTTP Method:

### Controller

- Controller:
- Controller Method:
- Evidence:

### Service

- Service Type:
- Service Method:
- Service Interface:
- Service Implementation:
- Evidence:

### ServiceImpl Method

- Method:
- Read Range:
- Evidence:

### Mapper Calls

ServiceImpl Method에서 직접 발견한 호출만 출력한다.

예:

- materialMapper.selectMaterial()
- materialMapper.insertMaterial()

직접 Mapper 호출이 없으면:

- Direct Mapper Call: NOT_FOUND

추가 검색하지 않는다.

### Execution Flow

예:

Controller.create()
→ MaterialService.create()
→ MaterialServiceImpl.create()
→ materialMapper.selectMaterial()
→ materialMapper.insertMaterial()
→ STOP

### Search Status

FOUND

또는

AMBIGUOUS

또는

NOT_FOUND

추가 분석을 수행하지 않는다.