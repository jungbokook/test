---
name: be-test-controller
description: Backend URL에서 Controller부터 Mapper Interface/Method까지 전체 파일 Read를 피하고 필요한 Method 범위만 부분 Read하여 빠르게 추적하는 성능 테스트 Skill.
argument-hint: "<HTTP Method|UNKNOWN> <Backend URL>"
allowed-tools: Read, Grep
---

# BE Fast Search Test - Mapper Partial Read

## 0. 목적

Backend URL 하나를 입력받아 다음까지만 추적한다.

Controller
→ Service
→ ServiceImpl
→ ServiceImpl 대상 Method 부분 Read
→ Mapper 호출 확인
→ Mapper Type 확인
→ Mapper Interface 위치 확인
→ Mapper Method 부분 확인
→ STOP

이번 테스트의 핵심은:

Mapper Interface 전체 파일을 읽지 않고
실제 호출된 Mapper Method만 부분적으로 확인했을 때
실행 시간이 얼마나 걸리는지 측정하는 것이다.

MyBatis XML과 SQL은 아직 분석하지 않는다.

---

# 1. 입력

사용법:

/be-test-controller <HTTP Method|UNKNOWN> <Backend URL>

예:

/be-test-controller POST /material/create

또는:

/be-test-controller UNKNOWN /material/create

---

# 2. 검색 범위

Backend 프로젝트:

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

절대경로를 출력하지 않는다.

---

# 3. 허용 도구

사용:

- Grep
- Read

사용하지 않는다:

- Glob
- Code Index
- Agent
- BE-REFERENCE

---

# 4. 핵심 성능 규칙

가장 중요한 규칙:

파일 전체 Read를 기본적으로 하지 않는다.

반드시 다음 순서로 처리한다.

정확한 Symbol 확인
→ Grep으로 위치 확인
→ 필요한 범위만 Read
→ 정보 확보
→ 즉시 다음 단계

규칙:

1. 동일 검색을 반복하지 않는다.
2. 이미 찾은 파일을 다시 찾지 않는다.
3. 이미 읽은 범위를 불필요하게 다시 읽지 않는다.
4. 현재 Backend 프로젝트부터 검색한다.
5. 찾지 못할 때만 다른 gipms-api-*로 확대한다.
6. Controller 확정 후 URL 검색을 중단한다.
7. Service 확정 후 Service 검색을 중단한다.
8. ServiceImpl 확정 후 구현체 검색을 중단한다.
9. Mapper Interface 확정 후 Mapper 검색을 중단한다.
10. Evidence는 최초 검색/Read 결과에서 확보한다.
11. Evidence를 위한 재검색을 하지 않는다.
12. 관련 있어 보인다는 이유로 다른 파일을 찾지 않는다.

내부적으로 다음 정보를 재사용한다.

KNOWN_FILES

KNOWN_SYMBOLS

VISITED_FILES

VISITED_METHODS

---

# STEP 1. URL 분리

입력 URL을 path 단위로 확인한다.

예:

/material/create

상위 path:

/material

마지막 segment:

/create

Controller 검색에서는 가능한 경우
식별력이 높은 상위 path를 먼저 사용한다.

---

# STEP 2. Controller 찾기

검색 범위:

gipms-api-*/**/*Controller.java

URL 상위 path를 먼저 Grep한다.

예:

/material

후보 Controller가 발견되면
전체 URL 검색을 중단한다.

후보 Controller만 확인한다.

상위 path만으로 확정할 수 없을 때만
마지막 segment를 보조 검색어로 사용한다.

---

# STEP 3. Controller Mapping 검증

Controller에서:

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

전체 URL과 HTTP Method가 일치하는 Method를 찾는다.

---

# STEP 4. HTTP Method 확정

입력이:

GET
POST
PUT
DELETE
PATCH

이면 Controller annotation과 일치해야 한다.

UNKNOWN이면 annotation에서 실제 Method를 확정한다.

동일 URL에 여러 HTTP Method가 존재하고
입력 Method가 UNKNOWN이면:

AMBIGUOUS

로 종료한다.

---

# STEP 5. Controller Method 확정

확인:

- Controller Class
- Controller Method
- HTTP Method
- Evidence

Controller Method가 확정되면
다른 Controller를 탐색하지 않는다.

---

# STEP 6. Service 호출 확인

Controller Method 내부에서 실제 호출되는 Service를 확인한다.

예:

materialService.create(request);

기록:

Service Variable:
materialService

Service Method:
create

---

# STEP 7. Service Type 확인

Controller에서 이미 확인한 field 또는 constructor를 이용한다.

예:

private final MaterialService materialService;

결과:

Service Type:
MaterialService

Service Variable:
materialService

Controller를 다시 검색하지 않는다.

---

# STEP 8. Service 찾기

Service 파일 경로를 모르면
현재 Backend 프로젝트에서 정확한 Type만 검색한다.

예:

interface MaterialService

현재 프로젝트에서 발견되면
다른 프로젝트를 검색하지 않는다.

현재 프로젝트에서 찾지 못한 경우에만
다른 gipms-api-*로 확대한다.

---

# STEP 9. Service Method 확인

Controller에서 실제 호출한 Method 선언만 확인한다.

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

먼저 현재 Backend 프로젝트에서만 찾는다.

구현체가 발견되면 검색을 중단한다.

ServiceImpl 파일 전체를 읽지 않는다.

파일 경로만 확보한다.

---

# STEP 11. ServiceImpl Method 위치 찾기

ServiceImpl 파일 하나에서만
정확한 Method 이름을 Grep한다.

예:

create(

가능하면 Method 선언을 우선 찾는다.

예:

public Result create(

Service Interface signature와 비교하여
정확한 구현 Method를 확정한다.

---

# STEP 12. ServiceImpl 부분 Read

Method 시작 line을 찾은 뒤
해당 위치부터 약 80줄만 Read한다.

예:

Method 시작:

420

초기 Read:

420 ~ 500

Method가 종료되면 추가 Read하지 않는다.

80줄 안에서 종료되지 않은 경우에만:

501 ~ 580

처럼 다음 범위를 읽는다.

Method 종료가 확인되면 즉시 중단한다.

ServiceImpl 전체 파일을 읽지 않는다.

---

# STEP 13. Mapper 호출 확인

ServiceImpl 대상 Method 범위에서
직접 호출되는 Mapper Method를 확인한다.

예:

materialMapper.selectMaterial(param);

materialMapper.insertMaterial(param);

기록:

- materialMapper.selectMaterial()
- materialMapper.insertMaterial()

조건문 안의 직접 Mapper 호출도 기록한다.

Local/private Method 내부는 현재 테스트에서 추적하지 않는다.

다른 Service 내부도 추적하지 않는다.

---

# STEP 14. Mapper Variable Type 확인

중요:

Mapper Type을 찾기 위해
ServiceImpl 전체 파일을 Read하지 않는다.

우선 이미 확보한 ServiceImpl 정보에서
Mapper field 또는 constructor 정보가 보이는지 확인한다.

예:

private final MaterialMapper materialMapper;

이면:

Mapper Variable:
materialMapper

Mapper Type:
MaterialMapper

---

# STEP 15. Mapper Type 정보가 현재 범위에 없는 경우

Mapper Variable Type이 현재 확보한 범위에 없을 때만
확정된 ServiceImpl 파일 하나에서
Mapper Variable 이름을 Grep한다.

예:

materialMapper

목표는 field/constructor declaration을 찾는 것이다.

예:

private final MaterialMapper materialMapper;

Type을 확인하면 즉시 검색을 중단한다.

ServiceImpl 전체 Read는 하지 않는다.

동일 variable을 반복 검색하지 않는다.

---

# STEP 16. Mapper Interface 위치 찾기

Mapper Type이:

MaterialMapper

로 확정되었다고 가정한다.

먼저 현재 Backend 프로젝트에서
정확한 Mapper Type 선언만 Grep한다.

예:

interface MaterialMapper

또는:

public interface MaterialMapper

검색 목표는 오직 Mapper Interface 파일 경로 확보다.

Mapper 관련 전체 검색을 하지 않는다.

현재 Backend 프로젝트에서 발견되면
다른 gipms-api-* 프로젝트를 검색하지 않는다.

현재 프로젝트에서 찾지 못한 경우에만
검색 범위를 확대한다.

Mapper 파일이 발견되면:

KNOWN_FILES에 저장한다.

---

# STEP 17. Mapper 파일 전체 Read 금지

Mapper Interface 경로를 확보했더라도
Mapper 파일 전체를 Read하지 않는다.

ServiceImpl에서 실제 호출한 Mapper Method 이름을 이용한다.

예:

insertMaterial

Mapper 파일 하나에서만:

insertMaterial

을 Grep한다.

목표:

Mapper Method 선언 line 찾기.

---

# STEP 18. Mapper Method 부분 확인

Mapper Method 위치가 발견되면
해당 위치 주변의 필요한 최소 범위만 Read한다.

Mapper Interface Method는 일반적으로 짧으므로
Method 위치 주변 약 10~20줄만 확인한다.

예:

line 75에서 발견

Read:

70 ~ 90

확인:

- Mapper Interface
- Mapper Method
- Parameter
- Return Type
- Evidence

Mapper 전체 파일은 읽지 않는다.

---

# STEP 19. 여러 Mapper Method

ServiceImpl Method에서 같은 Mapper의 여러 Method를 호출하면:

예:

materialMapper.selectMaterial()

materialMapper.insertMaterial()

Mapper Interface 파일은 한 번만 찾는다.

그 다음 동일 파일에서:

selectMaterial

insertMaterial

위치만 각각 확인한다.

Mapper 파일 경로를 다시 검색하지 않는다.

---

# STEP 20. 여러 Mapper Type

예:

materialMapper.selectMaterial()

historyMapper.insertHistory()

이면:

MaterialMapper

HistoryMapper

각 Type을 한 번씩만 확인한다.

각 Mapper Interface도 한 번씩만 찾는다.

동일 Mapper Type을 반복 검색하지 않는다.

---

# STEP 21. 조건 분기

ServiceImpl Method에서:

if (exists) {
    materialMapper.updateMaterial(...);
} else {
    materialMapper.insertMaterial(...);
}

가 확인되면:

MaterialMapper.updateMaterial()

MaterialMapper.insertMaterial()

두 Method 모두 확인한다.

조건 자체의 데이터 출처는 추적하지 않는다.

---

# STEP 22. Evidence

Evidence는 다음에 대해서만 확보한다.

- Controller Method
- Service Method
- ServiceImpl Method
- Mapper Method

형식:

Project Root 기준 상대경로:라인범위

예:

gipms-api-material/src/main/java/.../MaterialController.java:42-58

gipms-api-material/src/main/java/.../MaterialService.java:15-18

gipms-api-material/src/main/java/.../MaterialServiceImpl.java:420-468

gipms-api-material/src/main/java/.../MaterialMapper.java:72-81

절대경로는 출력하지 않는다.

Evidence 확보를 위한 재검색은 하지 않는다.

---

# STEP 23. 강제 STOP

실제로 호출되는 Mapper Interface와 Mapper Method가
모두 확인되면 즉시 종료한다.

이후 Grep/Read를 수행하지 않는다.

절대 수행하지 않는다:

- MyBatis XML 검색
- Mapper namespace 검색
- statement id XML 검색
- resultMap 검색
- SQL 검색
- SELECT 분석
- INSERT 분석
- UPDATE 분석
- DELETE 분석
- include 검색
- Local/private Method 추적
- 다른 Service 내부 추적
- DB 분석
- Oracle Metadata
- SAP 검색
- RFC 검색
- 외부 API 검색
- BE-REFERENCE 읽기
- Markdown 문서 생성

Mapper Method 확인 이후 추가 탐색을 수행하면
현재 테스트 범위를 초과한 것이다.

---

# STEP 24. 출력

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

### Mapper

각 Mapper에 대해:

- Mapper Variable:
- Mapper Type:
- Mapper Interface:
- Mapper Method:
- Parameter:
- Return Type:
- Evidence:

### Execution Flow

예:

Controller.create()
→ MaterialService.create()
→ MaterialServiceImpl.create()
→ MaterialMapper.selectMaterial()
→ MaterialMapper.insertMaterial()
→ STOP

분기가 직접 존재하면:

Controller.create()
→ MaterialService.create()
→ MaterialServiceImpl.create()
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