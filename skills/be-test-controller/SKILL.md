---
name: be-test-service
description: Backend URL에서 Controller를 찾고 해당 Controller Method가 실제 호출하는 Service와 Service Method까지만 빠르게 추적하는 성능 테스트 Skill.
argument-hint: "<HTTP Method|UNKNOWN> <Backend URL>"
allowed-tools: Read, Grep
---

# BE Controller → Service Fast Search Test

## 목적

이 Skill은 Backend 전체 분석을 수행하지 않는다.

입력받은 Backend URL을 기준으로 다음 단계까지만 찾는다.

1. Controller 파일
2. Controller Method
3. HTTP Method
4. Controller Method에서 실제 호출하는 Service
5. Service Method
6. Service 선언 위치 또는 실제 구현 위치
7. Evidence

Service 이후 호출은 절대 추적하지 않는다.

다음은 분석하지 않는다.

- Mapper
- MyBatis XML
- SQL
- DB
- SAP
- RFC
- 외부 API
- 다른 Service 내부 호출
- BE-REFERENCE
- Markdown 문서 생성

목표는 Controller → Service 탐색에 걸리는 시간을 측정하는 것이다.

---

# 입력

사용법:

/be-test-service <HTTP Method|UNKNOWN> <Backend URL>

예:

/be-test-service POST /material/create

또는:

/be-test-service UNKNOWN /material/create

---

# 검색 범위

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

Frontend 프로젝트는 검색하지 않는다.

---

# 기본 성능 원칙

검색 범위는 좁게 유지한다.

같은 문자열을 반복 검색하지 않는다.

이미 찾은 파일을 다시 Grep하지 않는다.

파일 경로를 알고 있으면 Grep하지 말고 Read한다.

Code Index는 사용하지 않는다.

Glob은 사용하지 않는다.

사용 가능 도구:

- Grep
- Read

---

# STEP 1. Controller 찾기

## 1-1. URL 분리

입력 URL을 다음처럼 해석한다.

예:

/material/create

상위 path:

/material

마지막 segment:

/create

가능하면 상위 path를 먼저 사용한다.

---

## 1-2. Controller 후보 검색

다음 범위에서만 Grep한다.

gipms-api-*/**/*Controller.java

먼저 상위 path를 검색한다.

예:

/material

Controller 후보가 발견되면 Backend 전체를 추가 검색하지 않는다.

후보 Controller 파일만 Read한다.

---

## 1-3. 전체 URL 검증

Controller 파일에서:

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

입력 URL과 정확히 일치하는지 확인한다.

---

## 1-4. HTTP Method

입력 Method가 다음 중 하나이면:

GET
POST
PUT
DELETE
PATCH

Controller annotation과 일치해야 한다.

UNKNOWN이면 annotation에서 실제 Method를 확정한다.

동일 URL에 여러 HTTP Method가 존재하면 임의 선택하지 않는다.

AMBIGUOUS로 출력하고 종료한다.

---

# STEP 2. Controller → Service

Controller Method가 확정되면 해당 Method 내부만 분석한다.

Controller 전체 비즈니스 로직을 분석하지 않는다.

---

## 2-1. 실제 Service 호출 확인

Controller Method 내부에서 실제 호출되는 객체를 확인한다.

예:

materialService.create(request)

그러면 기록한다.

Service variable:

materialService

Service Method:

create

---

## 2-2. Service 타입 확인

Controller의 field 선언 또는 constructor parameter를 확인한다.

예:

private final MaterialService materialService;

또는:

@Autowired
private MaterialService materialService;

또는 생성자 주입:

MaterialController(MaterialService materialService)

타입을 확인한다.

예:

MaterialService

---

## 2-3. Service 위치 검색

Service 타입의 파일 경로를 이미 알고 있으면 바로 Read한다.

모르면 다음 순서로 찾는다.

### 우선순위 1

현재 Controller와 동일 Backend 프로젝트 내부에서만 Grep한다.

검색 대상:

*.java

검색 문자열:

interface MaterialService

또는:

class MaterialService

정확한 타입 이름을 우선 검색한다.

### 우선순위 2

인터페이스만 발견된 경우 실제 구현체가 필요한 경우에만:

implements MaterialService

를 동일 Backend 프로젝트에서 검색한다.

예:

MaterialServiceImpl implements MaterialService

후보가 하나면 해당 파일만 Read한다.

### 우선순위 3

동일 Backend 프로젝트에서 찾지 못한 경우에만
다른 gipms-api-* 프로젝트로 검색 범위를 확대한다.

처음부터 모든 gipms-api-*를 반복 검색하지 않는다.

---

# STEP 3. Service Method 확인

Service 파일 또는 구현체를 찾으면 Controller가 호출한 Method가 존재하는지만 확인한다.

예:

Controller:

materialService.create(request)

Service:

create(...)

해당 Method 위치와 Evidence를 기록한다.

---

# 중요: Service 구현체 처리

Service가 interface인 경우:

Controller
→ MaterialService
→ MaterialServiceImpl

까지 위치를 확인할 수 있다.

하지만 MaterialServiceImpl.create() 내부의 호출은 분석하지 않는다.

즉:

Controller
  ↓
Service interface
  ↓
ServiceImpl Method

여기서 STOP.

---

# 매우 중요한 STOP 규칙

Service Method 위치가 확정되는 즉시 모든 검색을 종료한다.

다음을 절대 수행하지 않는다.

- Service Method 내부 비즈니스 로직 분석
- 다른 Service 호출 추적
- Mapper 검색
- Mapper Method 검색
- MyBatis XML 검색
- SQL 검색
- DTO 상세 분석
- DB 분석
- SAP/RFC 검색
- 외부 API 검색
- 관련 파일 추가 탐색
- BE-REFERENCE 읽기
- Markdown 문서 생성

Service 이후로 내려가면 이 테스트는 실패한 것이다.

---

# 중복 검색 방지

분석 중 다음 정보를 내부적으로 유지한다.

KNOWN_FILES

이미 발견한 파일 경로.

KNOWN_SYMBOLS

이미 발견한:

- Controller
- Controller Method
- Service Type
- Service Method
- ServiceImpl

동일 정보를 다시 Grep하지 않는다.

---

# Evidence

Evidence는 파일을 처음 Read할 때 같이 확보한다.

Evidence를 얻기 위한 두 번째 전체 검색을 하지 않는다.

절대경로를 출력하지 않는다.

Project Root 기준 상대경로만 사용한다.

예:

gipms-api-material/src/main/java/.../MaterialController.java:42-58

gipms-api-material/src/main/java/.../MaterialService.java:15-18

gipms-api-material/src/main/java/.../MaterialServiceImpl.java:71-90

---

# 출력

아래 형식으로만 출력한다.

## Controller → Service Search Result

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

### Flow

Controller.method()
→ Service.method()
→ ServiceImpl.method()
→ STOP

## Search Status

FOUND

또는

AMBIGUOUS

또는

NOT_FOUND

추가 분석을 수행하지 않는다.