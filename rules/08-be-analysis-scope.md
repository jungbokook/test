1---
name: be-analysis-scope
description: 선택한 Backend API를 기준으로 실제 실행 순서에 따라 Backend Business Logic, MyBatis/Oracle, RFC 및 기타 외부 연동을 추적하는 분석 범위를 정의한다.
---

# Backend Analysis Scope

## 1. 목적

Backend 분석은 사용자가 선택한 API 하나를 시작점으로
실제 Backend Source를 따라 전체 Business Logic의 실행 흐름을 분석한다.

단순한 계층 구조:

```text
Controller
→ Service
→ Mapper
→ SQL
```

만 출력하는 것이 목적이 아니다.

실제 Source에서 발생하는:

```text
Validation
조건/분기
데이터 조회
데이터 가공
상태 변경
내부 Method 호출
내부 Service 호출
DB 처리
RFC
REST API
SOAP
다른 Backend API
Message
File
기타 외부 시스템 연동
Exception
Response 생성
```

을 실제 실행 순서와 호출 관계에 따라 추적한다.


## 2. 입력

Backend 분석의 시작점은 사용자가 선택한 API이다.

입력 대상은 다음 중 하나일 수 있다.

```text
API ID
예: API-001

또는

HTTP Method + Backend URL
예: POST /api/equipment/search
```

API ID가 입력된 경우
해당 화면의 API 분석 문서를 먼저 확인하여
실제 HTTP Method와 Backend URL을 식별한다.

API 분석 문서는 탐색 시작점으로 사용한다.

Backend Business Logic의 최종 Evidence는
실제 Backend Source를 기준으로 한다.


## 3. 분석 시작점

분석 시작점은 실제 Controller Mapping이다.

기본 시작 흐름:

```text
API
 ↓
Controller Mapping
 ↓
Controller Method
 ↓
Service / Business Method
 ↓
실제 Business Logic
```

API 문서에서 Controller가 이미 확인되어 있다면
전체 Backend 프로젝트를 처음부터 다시 탐색하지 않는다.

기존 API 분석 결과를 시작점으로 사용하고
실제 Source에서 다시 확인한다.


## 4. Backend Source 범위

Backend Source는 현재 Code Index MCP의
Project Root 아래에서 탐색한다.

Backend 프로젝트 기준:

```text
gipms-api-*
```

관련 없는 Backend 프로젝트를
무조건 전체 탐색하지 않는다.

API URL,
Controller,
Class,
Method,
호출 관계,
실제 Source Evidence를 기준으로
필요한 프로젝트와 Source만 추적한다.

Source Path는 Project Root 기준 상대경로로 기록한다.

사용자에게 절대경로를 요청하지 않는다.

절대경로를 추측하거나 생성하지 않는다.


## 5. 실제 실행 흐름 기준 분석

Backend 분석은 파일 종류나 Layer 순서가 아니라
실제 Source의 실행 흐름을 기준으로 한다.

잘못된 방식:

```text
Controller
 ↓
Service
 ↓
Mapper
 ↓
SQL
 ↓
RFC
 ↓
Response
```

실제 Source가 다음과 같다면:

```text
Controller
 ↓
Service
 ↓
Validation
 ↓
DB SELECT
 ↓
조건 판단
 ↓
RFC 호출
 ↓
RFC 결과 처리
 ↓
DB UPDATE
 ↓
REST API 호출
 ↓
조건 판단
 ↓
DB INSERT
 ↓
Response 생성
```

최종 분석 역시 이 순서를 유지한다.


## 6. Business Logic 분석 범위

Business Logic에서는 실제 Source에서 확인되는
다음 내용을 분석한다.

```text
입력값 처리
Validation
조건문
분기
반복문
데이터 변환
값 계산
상태 변경
객체 생성
Parameter 생성
Request 생성
Response 변환
내부 Method 호출
내부 Service 호출
공통 Service 호출
Repository / Mapper 호출
외부 시스템 호출
Exception 발생 조건
Exception 처리
Return 값 생성
```

Method 이름만 보고 Business Logic을 추측하지 않는다.

실제 Method Body와 관련 Source를 확인한다.


## 7. 조건 / 분기 보존

조건문은 단순히:

```text
조건 확인
```

으로 축약하지 않는다.

실제 Source에서 확인 가능한 경우
조건과 분기 결과를 연결해서 기록한다.

예:

```text
equipmentType == "A" ?
 │
 ├─ YES
 │   ↓
 │   RFC #1 호출
 │   ↓
 │   RFC Response 처리
 │
 └─ NO
     ↓
     Local Data 사용
```

다중 조건도 실제 구조를 유지한다.

예:

```text
status == "READY"
AND
plantCode != null
        │
        ├─ TRUE
        │    ↓
        │   다음 처리
        │
        └─ FALSE
             ↓
            Exception
```

조건을 실제 Source Evidence 없이
업무 의미로 확대 해석하지 않는다.


## 8. 내부 Method 호출 추적

Business Logic 중 내부 Method가 호출되면
현재 기능의 실행 흐름에 영향을 주는 경우 추적한다.

예:

```text
saveEquipment()
        ↓
validateEquipment()
        ↓
createRequest()
        ↓
callExternalSystem()
        ↓
convertResponse()
```

Private Method라고 해서 생략하지 않는다.

현재 기능의:

```text
조건
데이터 변환
DB 처리
외부 연동
Exception
Response
```

에 영향을 주는 Method는 분석 대상이다.


## 9. 내부 Service / 공통 Service 추적

Business Logic 중 다른 Service 또는 공통 Service가 호출될 수 있다.

예:

```text
EquipmentService
        ↓
CommonService
        ↓
InterfaceService
```

현재 Project Root 아래 실제 Source가 존재하고
현재 기능 실행 흐름에 직접 참여한다면 계속 추적한다.

단순히:

```text
CommonService 호출
```

이라고 끝내지 않는다.

현재 기능과 관련된 실제 Method 내부의
Business Logic을 확인한다.


## 10. DB 처리는 호출 위치에서 분석

DB 처리를 Business Logic 마지막에 별도로 모으지 않는다.

실제 Mapper 호출 위치에 배치한다.

예:

```text
Business Logic
 │
 ├─ Step 1. 입력 검증
 │
 ├─ Step 2. DB #1
 │    └─ SELECT
 │
 ├─ Step 3. 결과 조건 판단
 │
 ├─ Step 4. RFC #1
 │
 ├─ Step 5. DB #2
 │    └─ UPDATE
 │
 └─ Step 6. Response 생성
```

DB 호출이 여러 번 존재하면
하나로 합치지 않는다.

각 DB 호출을 실제 실행 순서에 따라 구분한다.

예:

```text
DB #1
DB #2
DB #3
...
```


## 11. MyBatis 분석 범위

Mapper 호출이 발견되면
실제 MyBatis Mapping까지 추적한다.

분석 대상:

```text
Mapper Interface
Mapper Method
MyBatis Mapper XML
Namespace
Statement ID
Parameter Type
Parameter Mapping
Result Type / Result Map
SQL
Dynamic SQL
```

Dynamic SQL에서는 실제 Source에 존재하는 다음 요소를 분석한다.

```text
<if>
<choose>
<when>
<otherwise>
<foreach>
<where>
<set>
<trim>
<include>
<bind>
```

사용되지 않는 요소를 임의로 추가하지 않는다.


## 12. SQL 분석 범위

실제 MyBatis Statement에서 실행되는 SQL을 분석한다.

SQL 유형:

```text
SELECT
INSERT
UPDATE
DELETE
MERGE
Stored Procedure 호출
기타 실제 SQL
```

분석 항목:

```text
SQL 목적
입력 Parameter
Parameter Mapping
사용 Table / View
사용 Column
JOIN
WHERE
조건
Subquery
정렬
집계
Dynamic SQL
INSERT 값
UPDATE 대상 값
DELETE 조건
MERGE 조건
Result Mapping
```

SQL을 단순히:

```text
DB 조회
```

라고 축약하지 않는다.


## 13. Oracle MCP 사용

Oracle MCP는 실제 DB Metadata 확인이 필요한 경우 사용한다.

확인 대상 예:

```text
Table
View
Column
Data Type
Nullable
Primary Key
DB Object 존재 여부
```

Source의 SQL과 Oracle Metadata를 교차 확인한다.

Oracle MCP는 Read-Only 분석 목적으로만 사용한다.

다음 작업은 수행하지 않는다.

```text
INSERT
UPDATE
DELETE
MERGE 실행
CREATE
ALTER
DROP
TRUNCATE
기타 DDL / DML 실행
```

Source에 UPDATE / DELETE / INSERT SQL이 존재하더라도
그 SQL을 실제 Oracle DB에 실행하지 않는다.


## 14. SQL 의미 판단

Method명이나 Mapper명만 보고
DB 작업 의미를 판단하지 않는다.

예:

```text
deleteEquipment()
```

라는 Method가 존재하더라도
실제 MyBatis SQL이:

```text
UPDATE EQUIPMENT
SET STATUS = 'D'
```

라면 실제 DB 동작은 물리 DELETE가 아니라
UPDATE 기반 상태 변경이다.

따라서 최종 분석은 실제 SQL을 기준으로 한다.


## 15. 외부 연동은 Business Logic의 일부

RFC 또는 기타 외부 연동을
Backend 처리 마지막 단계로 고정하지 않는다.

외부 연동은 Business Logic의 어느 위치에서든 발생할 수 있다.

예:

```text
Service
 ↓
DB SELECT
 ↓
RFC #1
 ↓
데이터 변환
 ↓
DB UPDATE
 ↓
REST API #1
 ↓
RFC #2
 ↓
DB INSERT
 ↓
Response
```

실제 Source가 이 순서라면
분석 결과도 동일한 순서를 유지한다.


## 16. 외부 연동 횟수 제한 금지

하나의 API 처리 중
동일하거나 서로 다른 외부 연동이 여러 번 발생할 수 있다.

예:

```text
RFC #1
RFC #2
REST #1
RFC #3
SOAP #1
다른 Backend API #1
```

여러 호출을 하나의:

```text
외부 시스템 연동
```

으로 합치지 않는다.

각 호출을 독립적으로 식별하고
실제 호출 위치에 배치한다.


## 17. 외부 연동 유형

실제 Source에서 발견되는 외부 연동을 분석한다.

예:

```text
SAP RFC
RFC
REST API
HTTP / HTTPS
SOAP
WebService
다른 Backend API
Gateway
Interface Server
Message Queue
Messaging
File Interface
Batch 연계
Socket
기타 외부 시스템 호출
```

위 목록에 없는 방식이라도
실제 Source에서 외부 시스템 경계가 확인되면
분석 대상에 포함한다.

연동 유형을 미리 가정하여
Source를 억지로 분류하지 않는다.


## 18. RFC 분석

RFC 호출이 발견되면
실제 Source에서 확인 가능한 범위까지 분석한다.

확인 항목:

```text
호출 위치
호출 Method
RFC Function
호출 조건
입력값 생성
Request Parameter
Request Mapping
RFC 호출
Response
Response Field
Response Mapping
결과 처리
Exception / Error 처리
후속 Business Logic
```

RFC 이름만 확인하고 종료하지 않는다.

예:

```text
Business Logic
        ↓
RFC Request 생성
        ↓
RFC Parameter Mapping
        ↓
RFC Function 호출
        ↓
RFC Response 수신
        ↓
Response Mapping
        ↓
결과 조건 판단
        ↓
다음 Business Logic
```


## 19. REST / HTTP 외부 연동 분석

외부 REST / HTTP 호출이 발견되면
실제 Source에서 확인 가능한 범위까지 분석한다.

확인 항목:

```text
호출 위치
대상 시스템
HTTP Method
URL
Header
Query Parameter
Path Parameter
Request Body
Request Mapping
호출 조건
Response
Response Mapping
Status 처리
Error 처리
Timeout 처리
후속 Business Logic
```

확인되지 않는 항목은 추측하지 않는다.


## 20. 기타 외부 연동 분석

SOAP / Message / File / 기타 연동도
동일한 원칙을 적용한다.

핵심은:

```text
무엇을 호출하는가
        ↓
언제 호출하는가
        ↓
어떤 데이터를 만드는가
        ↓
어떻게 전달하는가
        ↓
무엇을 받는가
        ↓
결과를 어떻게 변환하는가
        ↓
그 결과가 다음 Business Logic에 어떻게 사용되는가
```

를 실제 Source에서 확인하는 것이다.


## 21. 내부 Service 안의 외부 연동

외부 연동이 현재 Service에 직접 존재하지 않을 수 있다.

예:

```text
EquipmentService
        ↓
CommonService
        ↓
SapInterfaceService
        ↓
RFC Client
        ↓
SAP RFC
```

현재 Project Root 아래 실제 Source가 존재한다면
호출 관계를 따라 외부 시스템 호출 지점까지 추적한다.

최종 Execution Tree에서도 실제 계층을 유지한다.

예:

```text
EquipmentService.save()
        ↓
CommonService.getSapData()
        ↓
SapInterfaceService.call()
        ↓
RFC #1
        ↓
SAP
```


## 22. 외부 연동 후 Business Logic 계속 추적

RFC / REST / 기타 외부 호출이 끝났다고 해서
Backend 분석을 종료하지 않는다.

외부 연동 결과가 이후 처리에 사용될 수 있다.

예:

```text
RFC #1
 ↓
Response 수신
 ↓
Response Validation
 ↓
데이터 변환
 ↓
DB UPDATE
 ↓
조건 분기
 ↓
REST API #1
 ↓
Response 생성
```

외부 연동 이후 현재 API의 Business Logic이 계속되면
끝까지 추적한다.


## 23. 반복문 안의 DB / 외부 호출

반복문 내부에서 DB 또는 외부 시스템 호출이 발생할 수 있다.

예:

```text
for each equipment
        │
        ├─ DB SELECT
        │
        ├─ 조건 확인
        │
        ├─ RFC 호출
        │
        └─ DB UPDATE
```

이 경우 반복 구조를 제거하지 않는다.

단순히:

```text
RFC 여러 번 호출
```

이라고 축약하지 않는다.

실제 반복 조건과
반복 내부 처리 구조를 표현한다.


## 24. 예외 처리

Backend Business Logic에서 실제로 확인되는
Exception 흐름을 분석한다.

예:

```text
Validation 실패
        ↓
Exception 발생
```

```text
RFC Error
        ↓
Error Code 확인
        ↓
Business Exception 변환
```

```text
DB 결과 없음
        ↓
조건 판단
        ↓
Exception
```

Exception이 발생하는 조건과
가능한 경우 이후 처리 흐름을 연결한다.

Exception 이름만 나열하지 않는다.


## 25. Transaction

실제 Source에서 Transaction Evidence가 확인되는 경우 분석한다.

예:

```text
@Transactional
```

확인 가능한 경우:

```text
Transaction 범위
DB 변경 작업
Exception 발생 지점
Rollback과 관련된 명시적 설정
```

을 기록한다.

단, Framework 기본 동작만으로
확인되지 않은 Transaction 동작을 추측하지 않는다.

특히 외부 RFC / REST 호출과 DB 변경이
같은 Business Logic에 존재하는 경우
실제 실행 순서를 명확히 기록한다.


## 26. Response 생성

Business Logic 완료 후
최종 Response가 어떻게 만들어지는지 확인한다.

확인 항목:

```text
Service Return
DTO 변환
Response 객체 생성
Result Mapping
Controller Return
```

API Contract 단계에서 확인한 Response 구조와
실제 Backend Business Logic의 결과 생성 과정을 연결한다.


## 27. 실제 실행 Tree

최종 Backend 분석에서는
실제 실행 순서를 보존한 Execution Tree를 생성해야 한다.

예:

```text
POST /api/equipment/save
        ↓
Controller
        ↓
Service
        │
        ├─ 1. Validation
        │
        ├─ 2. DB #1 - SELECT
        │
        ├─ 3. 조건 판단
        │      │
        │      ├─ YES
        │      │    ↓
        │      │   RFC #1
        │      │
        │      └─ NO
        │           ↓
        │          Local 처리
        │
        ├─ 4. 데이터 변환
        │
        ├─ 5. DB #2 - UPDATE
        │
        ├─ 6. REST #1
        │
        ├─ 7. RFC #2
        │
        ├─ 8. 조건 판단
        │      │
        │      ├─ 성공 → DB #3 - INSERT
        │      └─ 실패 → Exception
        │
        └─ 9. Response 생성
        ↓
Controller Response
```

실제 Source에 없는 Step을
문서 형식을 맞추기 위해 추가하지 않는다.


## 28. 호출 개수 축약 금지

다음 항목은 실제 호출이 여러 개라면
모두 식별한다.

```text
Internal Method
Internal Service
Mapper
SQL
DB
RFC
REST API
SOAP
Message
File
기타 외부 연동
```

예:

```text
DB #1
DB #2
RFC #1
DB #3
REST #1
RFC #2
```

을:

```text
DB 처리
외부 연동
```

처럼 하나로 축약하지 않는다.


## 29. Source Evidence

주요 분석 결과에는 가능한 경우
다음 Evidence를 기록한다.

```text
Source Path
Class
Method / Function
Statement ID
Line Range
```

DB의 경우:

```text
Mapper Interface
Mapper XML
Namespace
Statement ID
SQL
Oracle Metadata
```

외부 연동의 경우:

```text
호출 Source
호출 Method
Client / Adapter / Interface
RFC Function 또는 External Endpoint
Request / Response Mapping Source
```

를 가능한 범위에서 기록한다.

Line Range를 확인할 수 없는 경우
임의로 생성하지 않는다.

```text
Line Range: 확인되지 않음
```

으로 기록한다.


## 30. Code Index의 역할

Code Index MCP는 다음 용도로 사용한다.

```text
Source 탐색
Symbol 탐색
Reference 탐색
Caller / Callee 확인
관련 Source 범위 축소
```

Code Index 결과만으로
Business Logic을 확정하지 않는다.

Business Logic은 실제 Source를 읽고 판단한다.


## 31. JAR / 외부 Dependency 경계

현재 Project Root 아래에 실제 Source가 존재하면
공통 프로젝트나 내부 모듈도 추적할 수 있다.

예:

```text
gipms-api-common
gipms-api-interface
기타 실제 Source 프로젝트
```

하지만 다음 영역으로 자동 확장하지 않는다.

```text
*.jar
JAR 내부 Class
Decompiled Class
Maven Repository
.m2/
Gradle Cache
.gradle/
Project Root 외부 Dependency Source
```

외부 Library 호출 지점까지만 확인 가능한 경우:

```text
현재 Project Source
        ↓
External Library Method 호출
        ↓
════════ STOP ════════
JAR 내부 구현 분석 안 함
```

으로 처리한다.

외부 Library 내부 구현을 추측하지 않는다.


## 32. 미확인 항목

실제 Source 또는 Oracle Metadata에서
확인되지 않은 내용을 추측하지 않는다.

다음과 같이 기록한다.

```text
확인되지 않음
```

또는:

```text
현재 Project Source에서 확인되지 않음
```

호출 이름이나 일반적인 구현 패턴을 근거로
확인되지 않은 Business Logic을 만들어내지 않는다.


## 33. 안전 원칙

Backend 분석은 Read-Only 분석을 기본으로 한다.

실제 업무 데이터를 변경할 수 있는 작업을 실행하지 않는다.

특히:

```text
Oracle DML 실행
Oracle DDL 실행
RFC 실제 업무 처리 실행
외부 REST 변경 요청 실행
Message 발행
File 전송
실제 저장/수정/삭제 API 실행
```

등의 State-changing 작업은
분석 목적으로 실행하지 않는다.

Source를 분석하여 동작을 설명하는 것과
실제 업무 처리를 실행하는 것을 구분한다.


## 34. 출력 형식 원칙

최종 BE 문서는 FE 분석 문서와 동일하게
Excel의 단일 셀에 주요 섹션을 복사할 수 있는 구조를 사용한다.

따라서 주요 분석 결과는:

```text
독립적인 Text Block
+
ASCII Execution Tree
```

형식을 기본으로 한다.

각 상세 Block은
해당 Block만 따로 복사해도 의미를 이해할 수 있어야 한다.

다음과 같은 표현을 최소화한다.

```text
위와 동일
앞의 내용 참고
상기 조건 참고
앞에서 설명한 값
```

각 Block에 필요한 Context를 자체 포함한다.

세부 출력 모양은
`BE-REFERENCE.md`에서 정의한다.


## 35. 분석 완료 조건

현재 API에서 실제로 도달 가능한 Backend 흐름을
Source Evidence 기준으로 추적한 후 종료한다.

분석 완료 판단 기준:

```text
Controller 확인
        ↓
Business Logic 추적
        ↓
내부 Method / Service 추적
        ↓
DB 호출이 있으면 MyBatis / SQL 확인
        ↓
필요한 Oracle Metadata 확인
        ↓
RFC / 외부 연동이 있으면 실제 위치에서 추적
        ↓
외부 연동 이후 Business Logic 계속 추적
        ↓
Exception 흐름 확인
        ↓
Response 생성 확인
        ↓
Source Evidence 정리
        ↓
STOP
```


## 36. STOP

현재 선택한 API의 Backend Business Logic 분석이 완료되면 STOP 한다.

다음 기능으로 자동 확장하지 않는다.

```text
다른 API 분석
다른 화면 분석
관련 없는 Service 분석
관련 없는 Mapper 분석
관련 없는 DB Table 전체 분석
프로젝트 전체 RFC 분석
프로젝트 전체 외부 연동 분석
프로젝트 전체 Business Logic 분석
```

현재 선택한 API에서 실제 호출 관계로 연결되는 범위만 분석한다.

분석 완료 후
사용자가 다음 대상을 선택할 때까지 대기한다.