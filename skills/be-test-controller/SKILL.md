---
name: be-test-controller
description: Backend URL과 HTTP Method를 기준으로 Controller와 실제 매핑 Method만 빠르게 찾는 성능 테스트 Skill.
argument-hint: "<HTTP Method|UNKNOWN> <Backend URL>"
allowed-tools: Read, Grep
---

# BE Controller Fast Search Test

## 목적

이 Skill은 Backend 전체 분석을 수행하지 않는다.

오직 입력받은 Backend URL에 대응하는:

1. Controller 파일
2. Controller Method
3. HTTP Method
4. Evidence

만 찾고 즉시 종료한다.

Service, ServiceImpl, Mapper, XML, SQL, 외부 연동은 절대 분석하지 않는다.

---

# 입력

사용법:

/be-test-controller <HTTP Method|UNKNOWN> <Backend URL>

예:

/be-test-controller POST /material/create

또는:

/be-test-controller UNKNOWN /material/create

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

# 검색 원칙

목표는 정확성뿐 아니라 검색 시간을 최소화하는 것이다.

## 1. URL 검색

입력 URL 전체 문자열을 무조건 처음부터 검색하지 않는다.

예:

/material/create

이면 먼저 마지막 path segment:

create

를 Controller Java 파일에서 검색한다.

검색 대상은 반드시:

gipms-api-*/**/*Controller.java

범위로 제한한다.

예상 annotation:

@RequestMapping
@GetMapping
@PostMapping
@PutMapping
@DeleteMapping
@PatchMapping

---

## 2. 후보 Controller 확인

Grep 결과에서 URL과 관련된 Controller 후보를 찾는다.

후보 파일을 찾으면 해당 Controller 파일만 Read 한다.

Controller의 class-level mapping과 method-level mapping을 조합해서
실제 Backend URL과 일치하는지 확인한다.

예:

@RequestMapping("/material")

+

@PostMapping("/create")

=

/material/create

---

## 3. HTTP Method 확인

입력 Method가 POST/GET/PUT/DELETE/PATCH이면
해당 Method와 일치해야 한다.

입력이 UNKNOWN이면 Controller annotation에서 실제 Method를 확정한다.

동일 URL에 여러 HTTP Method가 존재하면 임의 선택하지 않는다.

발견한 Method들을 출력하고 종료한다.

---

# 매우 중요한 STOP 규칙

정확한 Controller와 Method가 확정되는 즉시 검색을 중단한다.

다음을 절대 수행하지 않는다.

- Service 검색
- ServiceImpl 검색
- Mapper 검색
- XML 검색
- SQL 검색
- DTO 전체 분석
- 호출 체인 분석
- 다른 관련 Controller 탐색
- BE-REFERENCE 읽기
- Markdown 문서 생성

---

# 출력

아래 형식으로만 출력한다.

## Controller Search Result

- URL:
- HTTP Method:
- Controller:
- Controller Method:
- Evidence:

## Search Status

FOUND

또는

AMBIGUOUS

또는

NOT_FOUND

추가 분석을 수행하지 않는다.