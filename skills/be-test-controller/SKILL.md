---
name: be-test-controller
description: Backend URL에서 Controller부터 실제 호출 Mapper Method까지 좁은 범위로 빠르게 추적하는 BE 탐색 성능 테스트 Skill.
argument-hint: "<HTTP Method|UNKNOWN> <Backend URL>"
allowed-tools: Read, Grep
---

# BE Fast Search Test

## 0. 목적

이 Skill은 최종 BE 분석 문서를 생성하는 Skill이 아니다.

Backend URL 하나를 입력받아 실제 실행 경로를 다음 범위까지만 추적한다.

Controller
→ Service
→ ServiceImpl
→ Mapper Interface
→ Mapper Method
→ STOP

현재 단계의 목적은 검색 속도를 측정하는 것이다.

분석 품질을 위해 실제 호출 관계는 유지하지만,
현재 단계에서 필요하지 않은 파일은 절대 탐색하지 않는다.

---

# 1. 입력

사용법:

/be-test-controller <HTTP Method|UNKNOWN> <Backend URL>

예:

/be-test-controller POST /material/create

또는:

/be-test-controller UNKNOWN /material/create

입력값:

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

Evidence는 Project Root 기준 상대경로를 사용한다.

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

현재 테스트에서는 Code Index보다 제한된 Grep 방식의
실측 속도가 더 빨랐으므로 Code Index를 사용하지 않는다.

---

# 5. 핵심 성능 규칙

검색 범위는 좁게 유지하고 호출 깊이는 현재 STOP 지점까지만 추적한다.

반드시 다음 규칙을 따른다.

1. 경로를 알고 있는 파일은 검색하지 말고 바로 Read한다.
2. 같은 symbol을 두 번 Grep하지 않는다.
3. 같은 파일을 찾기 위해 반복 검색하지 않는다.
4. 이미 읽은 파일은 필요한 정보가 확보되었다면 다시 읽지 않는다.
5. Controller가 발견되면 URL 전체 검색을 중단한다.
6. Service가 발견되면 Service 검색을 중단한다.
7. Mapper가 발견되면 Mapper 검색을 중단한다.
8. 현재 Backend 프로젝트에서 먼저 찾는다.
9. 찾지 못했을 때만 다른 gipms-api-* 프로젝트로 범위를 확대한다.
10. Evidence는 최초 Read 시 같이 확보한다.
11. Evidence를 얻기 위한 두 번째 검색을 하지 않는다.
12. 관련 있어 보인다는 이유만으로 추가 파일을 탐색하지 않는다.

내부적으로 다음 정보를 유지한다.

KNOWN_FILES

KNOWN_SYMBOLS

VISITED_FILES

VISITED_METHODS

동일 항목을 다시 탐색하지 않는다.

---

# STEP 1. Backend URL 분석

입력 URL을 path 단위로 나눈다.

예:

/material/create

상위 path:

/material

마지막 segment:

/create

Controller 검색 시 마지막 segment부터 무조건 검색하지 않는다.

가능하면 식별력이 높은 상위 path를 먼저 사용한다.

예:

/material/create

이면 우선:

/material

을 사용한다.

---

# STEP 2. Controller 후보 찾기

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

URL의 상위 path를 우선 검색한다.

예:

/material

후보 Controller가 발견되면 Backend 전체 URL 검색을 즉시 중단한다.

후보 파일만 Read한다.

후보가 너무 많거나 상위 path로 찾지 못한 경우에만
마지막 segment를 보조 검색어로 사용한다.

처음부터 여러 검색어로 반복 Grep하지 않는다.

---

# STEP 3. Controller Mapping 검증

후보 Controller 파일에서 다음을 확인한다.

1. class-level mapping
2. method-level mapping
3. HTTP Method

예:

@RequestMapping("/material")

+

@PostMapping("/create")

=

POST /material/create

입력 Backend URL과 전체 조합이 일치해야 한다.

---

# STEP 4. HTTP Method 확정

입력 Method가 다음 중 하나이면:

GET
POST
PUT
DELETE
PATCH

Controller annotation의 Method와 일치하는지 확인한다.

입력이 UNKNOWN이면 Controller annotation을 기준으로 실제 Method를 확정한다.

예:

@PostMapping("/create")

이면:

HTTP Method = POST

동일 URL에 서로 다른 HTTP Method가 여러 개 존재하고
입력 Method가 UNKNOWN이면 임의 선택하지 않는다.

Search Status:

AMBIGUOUS

로 출력하고 종료한다.

---

# STEP 5. Controller Method 확정

Backend URL과 HTTP Method가 일치하는
Controller Method 하나를 확정한다.

확인:

- Controller Class
- Controller Method
- HTTP Method
- Mapping
- Method parameter
- Evidence

Controller의 다른 Method는 분석하지 않는다.

Controller Method가 확정되면
다른 Controller 탐색을 즉시 중단한다.

---

# STEP 6. Controller → Service 호출 확인

확정된 Controller Method 내부만 분석한다.

Controller 전체를 분석하지 않는다.

실제로 호출되는 Service를 확인한다.

예:

materialService.create(request);

확인:

Service Variable:

materialService

Service Method:

create

Controller Method 내부에서 실제 실행되지 않는 객체는 무시한다.

---

# STEP 7. Service Type 확인

Controller의 field 또는 constructor injection을 확인한다.

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

이미 Controller 파일을 Read한 상태라면
Service Type 확인을 위해 Controller를 다시 검색하지 않는다.

---

# STEP 8. Service 위치 찾기

Service 파일 경로를 이미 알고 있으면 바로 Read한다.

모르면 현재 Controller가 위치한 동일 gipms-api-* 프로젝트에서 먼저 찾는다.

정확한 타입 이름을 검색한다.

예:

interface MaterialService

또는:

class MaterialService

검색 범위는 현재 Backend 프로젝트의 Java source로 제한한다.

현재 프로젝트에서 찾지 못한 경우에만
다른 gipms-api-* 프로젝트로 범위를 확대한다.

---

# STEP 9. Service Method 확인

Controller에서 실제 호출한 Service Method가
Service에 선언되어 있는지 확인한다.

예:

Controller:

materialService.create(request);

Service:

create(...)

확인:

- Service Interface/Class
- Service Method
- Parameter
- Return Type
- Evidence

Service의 다른 Method는 분석하지 않는다.

---

# STEP 10. ServiceImpl 찾기

Service가 interface인 경우에만 구현체를 찾는다.

예:

MaterialService

검색:

implements MaterialService

먼저 현재 Backend 프로젝트 내부에서만 검색한다.

예:

MaterialServiceImpl implements MaterialService

구현체 후보가 하나이면 해당 파일만 Read한다.

현재 프로젝트에서 찾지 못한 경우에만
다른 gipms-api-* 프로젝트로 검색 범위를 확대한다.

구현체가 여러 개이면 임의 선택하지 않는다.

실제 injection 구조를 확인할 수 있으면 확인하고,
확정할 수 없으면 AMBIGUOUS로 표시한다.

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

해당 Method의 시작과 종료 범위를 확인한다.

ServiceImpl 전체 Class를 분석하지 않는다.

다른 Method로 이동하지 않는다.

단, 현재 Method 내부에서 Mapper를 호출하기 전에
동일 Class의 private/local Method를 호출하고
그 Method 안에서 Mapper가 호출되는 경우에는
실제 호출 경로이므로 해당 local Method만 추적할 수 있다.

관련 없어 보이는 Method는 탐색하지 않는다.

---

# STEP 12. ServiceImpl → Mapper 호출 추출

확정된 ServiceImpl Method의 실제 실행 경로에서
Mapper 호출을 찾는다.

예:

materialMapper.selectMaterial(param);

materialMapper.insertMaterial(param);

확인:

- Mapper Variable
- Mapper Method
- 호출 조건
- 호출 순서

Mapper 호출이 여러 개이면 모두 기록한다.

예:

materialMapper.selectMaterial(...);

if (exists) {
    materialMapper.updateMaterial(...);
} else {
    materialMapper.insertMaterial(...);
}

실행 구조:

ServiceImpl.create()
├─ MaterialMapper.selectMaterial()
└─ IF exists
   ├─ YES → MaterialMapper.updateMaterial()
   └─ NO  → MaterialMapper.insertMaterial()

현재 단계에서는 조건의 상세 비즈니스 의미를 분석하지 않는다.

Mapper 호출 경로 보존만 수행한다.

---

# STEP 13. Mapper Type 확인

ServiceImpl의 field 또는 constructor parameter에서
Mapper Variable의 실제 Type을 확인한다.

예:

private final MaterialMapper materialMapper;

그러면:

Mapper Variable:

materialMapper

Mapper Type:

MaterialMapper

이미 ServiceImpl 파일을 Read했다면
Mapper Type을 찾기 위해 ServiceImpl을 다시 검색하지 않는다.

---

# STEP 14. Mapper Interface 찾기

Mapper 파일 경로를 알고 있으면 바로 Read한다.

모르면 현재 ServiceImpl과 동일 Backend 프로젝트에서
정확한 Mapper Type을 검색한다.

예:

interface MaterialMapper

또는:

public interface MaterialMapper

처음부터 모든 Mapper 파일을 검색하지 않는다.

현재 프로젝트에서 찾지 못한 경우에만
다른 gipms-api-* 프로젝트로 범위를 확대한다.

Mapper Interface가 발견되면 Mapper 검색을 즉시 중단한다.

---

# STEP 15. Mapper Method 확인

ServiceImpl에서 실제 호출한 Mapper Method가
Mapper Interface에 선언되어 있는지 확인한다.

예:

ServiceImpl:

materialMapper.insertMaterial(param);

Mapper:

int insertMaterial(MaterialParam param);

확인:

- Mapper Interface
- Mapper Method
- Parameter
- Return Type
- Evidence

Mapper의 다른 Method는 분석하지 않는다.

---

# STEP 16. 여러 Mapper 호출

하나의 ServiceImpl Method에서 여러 Mapper Method가 호출되면
실제로 호출되는 Mapper Method를 모두 확인한다.

예:

materialMapper.selectMaterial(...);

materialMapper.insertMaterial(...);

historyMapper.insertHistory(...);

결과:

ServiceImpl.create()
├─ MaterialMapper.selectMaterial()
├─ MaterialMapper.insertMaterial()
└─ HistoryMapper.insertHistory()

각 Mapper에 대해서 필요한 Interface와 Method까지만 확인한다.

---

# STEP 17. 조건 분기

Mapper 호출이 조건에 따라 달라지면 분기를 유지한다.

예:

if (exists) {
    materialMapper.updateMaterial(...);
} else {
    materialMapper.insertMaterial(...);
}

결과:

IF exists
├─ YES
│  └─ MaterialMapper.updateMaterial()
└─ NO
   └─ MaterialMapper.insertMaterial()

현재 단계에서는 조건을 만드는 DB 값이나
SQL 결과까지 내려가지 않는다.

---

# STEP 18. Local Method 처리

ServiceImpl Method가 같은 Class의 다른 Method를 호출하고
그 안에서 Mapper가 호출되는 경우 실제 실행 경로를 따라간다.

예:

create()
→ validate()
→ save()
→ materialMapper.insertMaterial()

이 경우:

create()
→ validate()
→ save()
→ Mapper

까지 추적할 수 있다.

단:

- 실제 호출된 local Method만 추적한다.
- 호출되지 않은 Method는 탐색하지 않는다.
- 다른 Service 호출은 현재 단계에서 깊게 분석하지 않는다.
- Mapper Method를 찾으면 현재 경로는 종료한다.

---

# STEP 19. Evidence

Evidence는 각 파일을 최초 Read할 때 확보한다.

형식:

Project Root 기준 상대경로:라인범위

예:

gipms-api-material/src/main/java/.../MaterialController.java:42-58

gipms-api-material/src/main/java/.../MaterialService.java:15-18

gipms-api-material/src/main/java/.../MaterialServiceImpl.java:71-96

gipms-api-material/src/main/java/.../MaterialMapper.java:20-27

절대경로는 출력하지 않는다.

Evidence 확보를 위해 동일 파일을 다시 전체 검색하지 않는다.

---

# STEP 20. STOP

실제로 호출되는 Mapper Interface와 Mapper Method가
모두 확인되는 즉시 분석을 종료한다.

여기서 반드시 STOP한다.

절대 수행하지 않는다.

- MyBatis XML 검색
- Mapper namespace 검색
- statement id 검색
- XML include 검색
- SQL 검색
- SELECT 분석
- INSERT 분석
- UPDATE 분석
- DELETE 분석
- DB Metadata 분석
- Oracle Metadata 분석
- SAP 검색
- RFC 검색
- 외부 API 검색
- Mapper 이후 처리 분석
- 관련 SQL 추측
- 관련 파일 추가 탐색
- BE-REFERENCE 읽기
- Markdown 문서 생성
- 전체 BE 분석

Mapper Method 이후로 내려가면
현재 성능 테스트의 범위를 초과한 것이다.

---

# STEP 21. 출력

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

### Mapper

각 실제 호출 Mapper에 대해:

- Mapper Type:
- Mapper Variable:
- Mapper Method:
- Parameter:
- Return Type:
- Evidence:

### Execution Flow

실제 호출 순서대로 표시한다.

예:

Controller.create()
→ MaterialService.create()
→ MaterialServiceImpl.create()
→ MaterialMapper.selectMaterial()
→ MaterialMapper.insertMaterial()
→ STOP

분기가 있으면:

Controller.create()
→ MaterialService.create()
→ MaterialServiceImpl.create()
→ MaterialMapper.selectMaterial()
→ IF exists
   ├─ YES → MaterialMapper.updateMaterial()
   └─ NO  → MaterialMapper.insertMaterial()
→ STOP

### Search Status

FOUND

또는

AMBIGUOUS

또는

NOT_FOUND

추가 분석을 수행하지 않는다.