---
name: be-test-controller
description: Backend URL에서 Controller, Service, ServiceImpl을 찾고 ServiceImpl Method 내부의 실제 Mapper 호출명까지만 확인하는 성능 테스트 Skill.
argument-hint: "<HTTP Method|UNKNOWN> <Backend URL>"
allowed-tools: Read, Grep
---

# BE Fast Search Test - Mapper Call Detection

## 0. 목적

이 Skill은 최종 BE 분석을 수행하지 않는다.

Backend URL 하나를 입력받아 다음 경로까지만 빠르게 확인한다.

Controller
→ Service
→ ServiceImpl
→ ServiceImpl Method
→ Mapper 호출명 확인
→ STOP

이번 테스트의 핵심 목적은:

ServiceImpl Method 내부에서 Mapper 호출명을 확인하는 작업 자체가
느린지 측정하는 것이다.

Mapper Interface 파일은 찾지 않는다.

Mapper Method 선언도 찾지 않는다.

MyBatis XML과 SQL도 찾지 않는다.

---

# 1. 입력

사용법:

/be-test-controller <HTTP Method|UNKNOWN> <Backend URL>

예:

/be-test-controller POST /material/create

또는:

/be-test-controller UNKNOWN /material/create

입력:

- HTTP Method
- Backend URL

---

# 2. 검색 대상

Backend 프로젝트만 검색한다.

대상:

gipms-api-*

Frontend 프로젝트는 검색하지 않는다.

---

# 3. 검색 제외

다음은 검색하지 않는다.

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

절대경로 기반 탐색도 하지 않는다.

Evidence는 Project Root 기준 상대경로만 사용한다.

---

# 4. 허용 도구

현재 테스트에서는 다음만 사용한다.

- Read
- Grep

사용하지 않는다.

- Code Index
- Glob
- Agent
- BE-REFERENCE

---

# 5. 최우선 성능 규칙

검색 범위는 최대한 좁게 유지한다.

반드시 다음 규칙을 따른다.

1. 파일 경로를 알고 있으면 Grep하지 말고 Read한다.
2. 동일 symbol을 두 번 Grep하지 않는다.
3. 동일 파일을 찾기 위해 반복 검색하지 않는다.
4. 이미 읽은 파일에서 얻을 수 있는 정보는 다시 검색하지 않는다.
5. Controller가 확정되면 URL 검색을 즉시 중단한다.
6. Service가 확정되면 Service 검색을 즉시 중단한다.
7. ServiceImpl이 확정되면 구현체 검색을 즉시 중단한다.
8. 현재 Backend 프로젝트에서 먼저 검색한다.
9. 현재 프로젝트에서 찾지 못한 경우에만 다른 gipms-api-*로 확대한다.
10. Evidence는 최초 Read 시 같이 확보한다.
11. Evidence를 얻기 위한 추가 검색을 하지 않는다.
12. 관련 있어 보인다는 이유로 다른 파일을 탐색하지 않는다.
13. Mapper 호출명을 찾은 이후 Grep을 실행하지 않는다.

내부적으로 다음을 기억한다.

KNOWN_FILES

KNOWN_SYMBOLS

VISITED_FILES

VISITED_METHODS

동일 항목을 다시 탐색하지 않는다.

---

# STEP 1. URL 분리

입력 Backend URL을 path 단위로 확인한다.

예:

/material/create

상위 path:

/material

마지막 segment:

/create

Controller 검색 시 가능한 경우 식별력이 높은 상위 path를 먼저 사용한다.

마지막 segment부터 무조건 검색하지 않는다.

---

# STEP 2. Controller 후보 검색

Controller Java 파일만 대상으로 Grep한다.

검색 범위:

gipms-api-*/**/*Controller.java

검색 대상:

Spring Mapping annotation의 URL 문자열

예:

@RequestMapping
@GetMapping
@PostMapping
@PutMapping
@DeleteMapping
@PatchMapping

우선 URL의 상위 path를 검색한다.

예:

/material

후보 Controller가 발견되면 URL에 대한 Backend 전체 검색을 중단한다.

후보 Controller 파일만 Read한다.

상위 path로 찾지 못했거나 후보를 확정할 수 없는 경우에만
마지막 segment를 보조 검색어로 사용한다.

---

# STEP 3. Controller Mapping 검증

후보 Controller 파일에서 다음을 확인한다.

- class-level mapping
- method-level mapping
- HTTP Method

예:

@RequestMapping("/material")

+

@PostMapping("/create")

=

POST /material/create

class-level mapping과 method-level mapping을 조합한 실제 URL이
입력 Backend URL과 일치해야 한다.

---

# STEP 4. HTTP Method 확정

입력 Method가:

GET
POST
PUT
DELETE
PATCH

중 하나이면 Controller annotation과 일치하는지 확인한다.

입력이 UNKNOWN이면 Controller annotation에서 실제 Method를 확정한다.

예:

@PostMapping("/create")

이면:

HTTP Method = POST

동일 URL에 여러 HTTP Method가 존재하고
입력 Method가 UNKNOWN이면 임의로 선택하지 않는다.

Search Status:

AMBIGUOUS

로 출력하고 종료한다.

---

# STEP 5. Controller Method 확정

URL과 HTTP Method가 일치하는 Controller Method를 확정한다.

확인:

- Controller Class
- Controller Method
- HTTP Method
- Mapping
- Evidence

Controller Method가 확정되면
다른 Controller를 추가 탐색하지 않는다.

Controller의 다른 Method도 분석하지 않는다.

---

# STEP 6. Controller Method에서 Service 호출 확인

확정된 Controller Method 내부만 확인한다.

예:

materialService.create(request);

실제 호출되는 Service만 기록한다.

확인:

Service Variable:

materialService

Service Method:

create

Controller 전체 Class의 다른 비즈니스 로직을 분석하지 않는다.

---

# STEP 7. Service Type 확인

이미 읽은 Controller 파일의:

- field
- constructor parameter
- injection declaration

중에서 Service Variable의 Type을 확인한다.

예:

private final MaterialService materialService;

또는:

@Autowired
private MaterialService materialService;

또는:

MaterialController(
    MaterialService materialService
)

확인:

Service Type:

MaterialService

Service Variable:

materialService

이 정보를 얻기 위해 Controller 파일을 다시 Grep하지 않는다.

---

# STEP 8. Service 위치 검색

Service 파일 경로를 이미 알고 있으면 바로 Read한다.

모르는 경우 현재 Controller가 위치한 동일 gipms-api-* 프로젝트에서만
정확한 Service Type을 검색한다.

예:

interface MaterialService

또는:

class MaterialService

현재 Backend 프로젝트에서 발견되면
다른 프로젝트를 검색하지 않는다.

현재 프로젝트에서 찾지 못한 경우에만
다른 gipms-api-* 프로젝트로 검색 범위를 확대한다.

---

# STEP 9. Service Method 확인

Controller에서 실제 호출한 Service Method가
Service에 선언되어 있는지만 확인한다.

예:

Controller:

materialService.create(request);

Service:

create(...)

확인:

- Service Type
- Service Method
- Parameter
- Return Type
- Evidence

Service의 다른 Method는 분석하지 않는다.

---

# STEP 10. ServiceImpl 검색

Service가 interface인 경우 실제 구현체를 찾는다.

예:

MaterialService

검색:

implements MaterialService

먼저 현재 Backend 프로젝트 내부에서만 검색한다.

예:

MaterialServiceImpl implements MaterialService

구현체가 하나이면 해당 파일만 Read한다.

현재 프로젝트에서 찾지 못한 경우에만
다른 gipms-api-* 프로젝트로 범위를 확대한다.

구현체가 여러 개이고 실제 구현체를 확정할 수 없으면
임의 선택하지 않는다.

AMBIGUOUS로 처리한다.

---

# STEP 11. ServiceImpl Method 확인

Controller에서 호출한 Service Method에 대응하는
ServiceImpl Method만 확인한다.

예:

Controller:

materialService.create(request);

ServiceImpl:

public Result create(Request request) {
    ...
}

확인:

- ServiceImpl Class
- ServiceImpl Method
- Method 범위
- Evidence

ServiceImpl 전체 Class를 분석하지 않는다.

다른 Method는 분석하지 않는다.

---

# STEP 12. Mapper 호출명만 추출

확정된 ServiceImpl Method 내부에서
직접 호출되는 Mapper 호출명만 확인한다.

예:

materialMapper.selectMaterial(param);

materialMapper.insertMaterial(param);

그러면 다음만 기록한다.

- materialMapper.selectMaterial()
- materialMapper.insertMaterial()

Mapper Variable의 Type은 찾지 않는다.

예:

materialMapper

가 발견되어도:

MaterialMapper

라는 실제 타입을 찾기 위한 추가 검색을 하지 않는다.

---

# STEP 13. 여러 Mapper 호출

ServiceImpl Method 안에서 Mapper 호출이 여러 개이면
직접 보이는 Mapper 호출명을 모두 기록한다.

예:

materialMapper.selectMaterial(param);

materialMapper.insertMaterial(param);

historyMapper.insertHistory(param);

결과:

- materialMapper.selectMaterial()
- materialMapper.insertMaterial()
- historyMapper.insertHistory()

각 Mapper의 Interface 파일은 찾지 않는다.

---

# STEP 14. 조건문 내부 Mapper

조건문 안에 직접 Mapper 호출이 있으면
호출명만 기록한다.

예:

if (exists) {

    materialMapper.updateMaterial(param);

} else {

    materialMapper.insertMaterial(param);

}

결과:

- materialMapper.updateMaterial()
- materialMapper.insertMaterial()

조건의 상세 비즈니스 의미는 분석하지 않는다.

조건에 사용되는 값의 출처도 추적하지 않는다.

---

# STEP 15. 현재 테스트에서 Local Method는 추적하지 않음

ServiceImpl Method에서 같은 Class의 local/private Method를 호출하더라도
현재 테스트에서는 그 Method 내부로 들어가지 않는다.

예:

public Result create(...) {

    validate(...);

    save(...);

}

현재 단계에서는:

validate()

save()

의 내부를 분석하지 않는다.

Mapper가 현재 ServiceImpl Method에 직접 보이는 경우만 기록한다.

이 규칙은 성능 병목 위치를 분리하기 위한 테스트용 규칙이다.

최종 BE 분석 규칙이 아니다.

---

# STEP 16. Mapper처럼 보이는 호출 판단

ServiceImpl Method 내부에서 다음 형태의 호출을 확인한다.

예:

materialMapper.selectMaterial(...)

historyMapper.insertHistory(...)

xxxDao.select(...)

xxxRepository.find(...)

프로젝트에서 사용하는 Mapper naming convention이 명확한 경우
그 naming을 기준으로 호출명을 기록할 수 있다.

하지만 실제 Type 확인을 위한 추가 검색은 하지 않는다.

확실하지 않은 호출은 추측해서 Mapper라고 단정하지 않는다.

---

# STEP 17. Evidence

현재 단계의 Evidence는 다음까지만 확보한다.

- Controller
- Service
- ServiceImpl

Evidence 형식:

Project Root 기준 상대경로:라인범위

예:

gipms-api-material/src/main/java/.../MaterialController.java:42-58

gipms-api-material/src/main/java/.../MaterialService.java:15-18

gipms-api-material/src/main/java/.../MaterialServiceImpl.java:71-96

Mapper Evidence는 현재 단계에서 만들지 않는다.

Mapper 파일 자체를 찾지 않기 때문이다.

절대경로는 출력하지 않는다.

---

# STEP 18. 강제 STOP

ServiceImpl Method 내부에서 직접 확인 가능한
Mapper 호출명을 추출하면 즉시 종료한다.

이 시점 이후 Grep을 실행하지 않는다.

절대 수행하지 않는다.

- Mapper Type 검색
- Mapper Interface 검색
- Mapper Java 파일 검색
- Mapper Method 선언 검색
- implements Mapper 검색
- @Mapper 검색
- MyBatis XML 검색
- namespace 검색
- statement id 검색
- XML include 검색
- SQL 검색
- SELECT 분석
- INSERT 분석
- UPDATE 분석
- DELETE 분석
- Local/private Method 추적
- 다른 Service 호출 추적
- DB 분석
- Oracle Metadata 분석
- SAP 검색
- RFC 검색
- 외부 API 검색
- BE-REFERENCE 읽기
- Markdown 문서 생성

Mapper 호출명 확인 이후 추가 탐색을 수행하면
현재 테스트는 실패한 것이다.

---

# STEP 19. 출력

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
- Service Variable:
- Service Method:
- Service Interface:
- Service Implementation:
- Evidence:

### Mapper Calls

ServiceImpl Method에서 직접 발견한 Mapper 호출명만 출력한다.

예:

- materialMapper.selectMaterial()
- materialMapper.insertMaterial()
- historyMapper.insertHistory()

Mapper 호출이 직접 보이지 않으면:

- Direct Mapper Call: NOT_FOUND

이라고 출력한다.

Mapper를 찾기 위한 추가 검색은 하지 않는다.

### Execution Flow

예:

Controller.create()
→ MaterialService.create()
→ MaterialServiceImpl.create()
→ materialMapper.selectMaterial()
→ materialMapper.insertMaterial()
→ STOP

조건 분기가 직접 보이는 경우:

Controller.create()
→ MaterialService.create()
→ MaterialServiceImpl.create()
→ IF exists
   ├─ YES → materialMapper.updateMaterial()
   └─ NO  → materialMapper.insertMaterial()
→ STOP

### Search Status

FOUND

또는

AMBIGUOUS

또는

NOT_FOUND

추가 분석을 수행하지 않는다.